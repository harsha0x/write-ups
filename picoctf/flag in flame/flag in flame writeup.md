picoCTF Walkthrough: Cracking the "Flag in Flame"
=================================================

**Category:** Forensics **Challenge:** Flag in Flame

I recently decided to take on the **Flag in Flame** challenge in picoCTF. It was a great exercise in recognizing data encoding and file signatures. It started with a simple file download but quickly turned into a multi-step decoding process. Here is exactly how I solved it.

### The Suspicious "Log" File

The challenge provided a single file named `logs.txt`. Typically, when you see a log file in a CTF, you expect to be grep-ing through lines of server activity or error messages.

However, the first thing that caught my eye was the **file size**. For a text file, it was unusually large. I opened it up to inspect the contents, and my suspicions were confirmed immediately. There were no timestamps, no IP addresses, and no readable words---just a massive, continuous block of alphanumeric characters.

### Recognizing the Encoding

I analyzed the character set. It consisted of `A-Z`, `a-z`, `0-9`, and symbols like `+` and `/`. This is the classic signature of **Base64 encoding**.

One detail stood out: usually, Base64 strings end with one or two equal signs (`=`) as padding. In this file, **there were no equal signs** at the end. While padding is common, its absence doesn't mean it's not Base64; it just means the data length happened to align perfectly with the encoding blocks. I was confident this was a binary file disguised as text.

### Decoding the Payload

I turned to the command line to decode it. My first instinct was to just check the output, so I ran the `base64 -d` command printing to the standard output.

This was a mistake! My terminal was instantly flooded with garbage characters and weird symbols. This confirmed that the decoded data wasn't a hidden text message (like a password), but rather a **binary file**---likely an image given the challenge name "Flag in Flame."

I ran the command again, this time redirecting the output to a file. I guessed it might be a JPEG:

Bash

```
base64 -d logs.txt > flag.jpg

```

I checked the folder, and sure enough, I had a valid image file.

### Visual Analysis and Hex Extraction

I opened `flag.jpg`. It was a picture of flames (fitting the theme), but I didn't need to run any complex steganography tools like `steghide` or `zsteg`.

I simply looked at the image data visually. Hidden within the image itself was a long string of text. I zoomed in and transcribed it. The string was composed entirely of numbers `0-9` and letters `A-F`.

This specific range of characters is the hallmark of **Hexadecimal encoding**.

### The Final Flag

I copied the hex string from the image and decode it into ASCII text. There are a few ways to do this, but I used the the online convertor this time:


The output was immediate. The hex string decoded perfectly into readable text, revealing the flag in the standard `picoCTF{...}` format.

### Summary

This challenge was all about trusting your instincts on file types.

1.  **Analyze:** The `logs.txt` was too big and looked like Base64 (even without padding).

2.  **Decode:** Converted the text blob into a binary `.jpg`.

3.  **Observe:** Found the Hex string visually hidden in the image.

4.  **Solve:** Decoded the Hex to get the flag.
