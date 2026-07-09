🔍 picoCTF Walkthrough: Verify
==============================

**Category:** Forensics / General Skills 📂 **Challenge:** Verify

Sometimes, the challenge isn't about breaking in---it's about finding the needle in the haystack once you're already inside. The **Verify** challenge was a perfect test of Linux command-line fu, involving SSH, checksums, and a bit of scripting analysis. Here is how I solved it.

### 🔐 Step 1: The Entry

The challenge provided SSH credentials (host, user, and password). I started by connecting to the remote machine:

Bash

```
ssh -p PORT ctf-player@rhea.picoctf.net

```

Once in, I looked around. I saw a bunch of files and a script called `decrypt.sh`.

### 📜 Step 2: Analyzing the Mechanism

Before blindly running things, I needed to understand what I was dealing with. I read the source code of the decryption script:

Bash

```
cat decrypt.sh

```

The script logic was straightforward: it was designed to decrypt a file, but it required a specific file as an input. The hint lay in the challenge description---I needed to find a file that matched a specific **SHA-256 checksum**.

The problem? There were hundreds of files in the `files` directory. I couldn't just guess which one was the true flag container.

### 🔎 Step 3: The Hunt for the Checksum

I knew the target checksum (provided in the challenge description), but I didn't know which file matched it.

I started by searching online for "how to find checksum of a file." I learned about the `sha256sum` command. But running this manually on every single file would take forever.

I needed a way to automate it. I dug a bit deeper into Linux commands and found two powerful options to pair with `sha256sum`: using `find` with `-exec`, or piping the output to `xargs`.

### ⚡ Step 4: Command Line Magic

I decided to use a command that would list all files in the folder and calculate their checksums in one go. I constructed a command like this:

Bash

```
sha256sum files/* | grep "THE_TARGET_CHECKSUM"

```

*(Or alternatively, using the `find` method I researched: `find files -type f -exec sha256sum {} + | grep "TARGET_CHECKSUM"`)*

The terminal churned through the list and---**bingo!** 🎯

The `grep` command filtered out the noise and left me with exactly one match. I now had the filename of the *real* encrypted flag.

### 🔓 Step 5: Decrypting the Prize

With the correct filename in hand, I went back to the `decrypt.sh` script I analyzed earlier. I ran the script, passing the file I just found as the argument:

Bash

```
./decrypt.sh files/target_file_name

```

The script did its magic, stripped away the encryption, and printed the flag right to the console: `picoCTF{...}`.

### 🚩 Conclusion

This challenge was a great exercise in using Linux utilities to sift through data. It taught me that knowing *how* to decrypt is only half the battle; knowing *what* to decrypt is just as important! 🐧💡
