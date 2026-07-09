🚪 picoCTF Walkthrough: Crack the Gate 1
========================================

**Category:** Web Exploitation 🌐 **Challenge:** Crack the Gate 1

Web challenges often hide the keys to the kingdom in places most users never look. **Crack the Gate 1** was a classic example of "security by obscurity," and here is how I managed to break in.

### 📧 Step 1: The Initial Recon

I started by reading the challenge description carefully. It gave me a specific target email to work with: `ctf-player@picoctf.org`.

Naturally, my first move was to try the front door. I went to the login form, entered the email, and tried a random password. As expected, the application rejected me immediately:

> ❌ **Invalid credentials**

The front door was locked, so it was time to look for a window.

### 🕵️‍♂️ Step 2: Snooping in the Source

When a web page blocks you, the next step is always to check what's happening behind the scenes. I right-clicked and hit **View Page Source** (or `Ctrl+U`).

I scrolled through the HTML, looking for comments or hidden scripts. That's when I stumbled upon something strange---a comment containing a string of weird, jumbled text. It didn't look like standard base64 or hex; the structure seemed like a simple substitution cipher.

### 🔄 Step 3: Decoding the Clue

I suspected **ROT13** (a Caesar cipher that rotates letters by 13 places), which is a CTF favorite for hiding hints.

I tossed the text into a decoder, and sure enough, it unscrambled into a readable sentence. The hidden message was a directive for developers:

> **"Use header X-Dev-Access: yes"** 🔓

This wasn't a password; it was an instruction to modify the HTTP request itself.

### 🛠️ Step 4: Intercepting with Burp Suite

Now that I knew the secret handshake, I needed a way to send it to the server. I fired up **Burp Suite** to act as a proxy between my browser and the server.

1.  I captured the login request again.

2.  I sent it to the **Repeater** tab.

3.  I manually added the new header to the request:

    HTTP

    ```
    X-Dev-Access: yes

    ```

4.  I hit **Send**.

### 🚩 Conclusion

The server response came back, and this time, there was no "Invalid credentials" error. Instead, the response contained the sweet victory text: `picoCTF{...}`.

This challenge was a great reminder that not all authentication happens in the password field---sometimes, it's hidden in the headers! 🚀
