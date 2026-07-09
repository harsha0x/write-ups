PicoCTF Write-up: Hidden in Plain Sight
=======================================

Challenge Description
---------------------

> **Category:** Forensics
>
> **Difficulty:** Easy
>
> **Description:** Sometimes the best place to hide something is right in front of your eyes. Look closely at the provided file to find the hidden flag.

* * * * *

🛠 Tools Used
-------------

-   **ExifTool:** For reading file metadata.

-   **Steghide/Binwalk:** For extracting hidden data from images.

* * * * *

🚩 Solution Walkthrough
-----------------------

### 1\. Initial File Analysis

First, we check the file type and basic metadata to see if any obvious comments or headers have been tampered with.

Bash

```
file hidden.jpg
exiftool hidden.jpg

```

*Not*Note: Look specifically for the "Comment" or "Artist" fields in the metadata. And decode it using base64 -d*


### 3\. Steganography Check

Many "Hidden in Plain Sight" challenges use **LSB (Least Significant Bit)** steganography. You can use an online tool or a command-line tool like `steghide`. The steghide password is still a base64 coded data.
Upon decoding it we get a pssword for the file that is hidden using steghide.

Bash

```
steghide extract -sf hidden.jpg
```
Using the above command we can access the hidden data with entering the password.

> This might be a simple or very easy challenge but something is better than nothing, I will solve some difficult challenges for now on. Happy Hacking
