⚔️ picoCTF Walkthrough: Crack the Gate 2
========================================

**Category:** Web Exploitation 🌐 **Challenge:** Crack the Gate 2

Brute forcing a login page is usually simple, but **Crack the Gate 2** threw a wrench in the gears. The challenge description warned that the login form would **lock out** any IP suspected of brute-forcing. I had the target email (`ctf-player@picoctf.org`) and a list of passwords, but I needed a way to beat the rate limiter. Here is how I smashed the gate.

### 🛑 Step 1: The Barrier

The application was smart. If I tried too many failed logins from my IP address, it would block me. To bypass this, I needed to make the server think the requests were coming from many *different* people, not just one attacker.

I researched HTTP headers used for identifying client IPs and found the golden key: `X-Forwarded-For`. This header tells the server the "original" IP of the client connecting through a proxy.

### 🎭 Step 2: Setting the Stage with Burp Suite

I fired up **Burp Suite** and captured a login request. I sent it to **Intruder** to automate the attack.

I needed to manipulate two things simultaneously:

1.  **The Password:** Iterating through the provided password list.

2.  **The IP Address:** Spoofing a new IP for every request to avoid the ban.

I added the header to my request: `X-Forwarded-For: 192.0.0.§1§`

### 💥 Step 3: The "Pitchfork" Attack

In the Intruder configuration, I set up the payloads.

-   **Payload 1 (IPs):** I set this to increment numbers (1, 2, 3...) so the header would become `192.0.0.1`, `192.0.0.2`, and so on.

-   **Payload 2 (Passwords):** I loaded the password list provided by the challenge.

By matching every password attempt with a fresh, fake IP address, I completely bypassed the brute-force protection. 🧠

### 🎯 Step 4: Spotting the Winner

I launched the attack and watched the results fly by. Most responses looked identical---failed logins.

However, I sorted the results by **Length**. One response stood out because it had a different file size compared to the others. I checked the response body for that specific request, and there it was---no error message, just the flag: `picoCTF{...}`.

### 🚩 Conclusion

This challenge was a masterclass in IP spoofing. It proved that rate limits based solely on IP addresses are easily bypassed if the server trusts the `X-Forwarded-For` header blindly! 🕵️‍♂️🚀
