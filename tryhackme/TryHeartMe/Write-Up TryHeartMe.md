---
layout: ../../layouts/BlogLayout.astro
title: "TryHeartMe THM write-up"
date: "FEB 15, 2026"
readTime: "5 MIN READ"
author: "Harsha"
category: "cyber"
---

## Introduction

**TryHeartMe** is an excellent, easy-difficulty web challenge featured in TryHackMe's Valentine-themed _Love at First Breach_ track. The challenge simulates a boutique online shop managed by Chief Inspector Valentine. Our primary objective? Find a way to purchase the hidden, restricted item **"Valenflag"** without having the necessary credits or permissions.

This room focuses heavily on client-side token security, specifically showcasing how misconfigured **JSON Web Tokens (JWT)** can lead to total authentication bypass and privilege escalation.

## The Attack Chain At-A-Glance

Plaintext

```
Create Account & Log In 
       ↓
Extract JWT from Browser Cookies
       ↓
Modify Payload (Change Role & Credits) + Set Alg to 'none'
       ↓
Inject Forged Token Back into Cookie
       ↓
Access Admin Panel & Purchase "Valenflag"
```

## Step-by-Step Breakdown

### Phase 1: Registration and Initial Reconnaissance

Upon navigating to `http://<MACHINE_IP>:5000`, we are greeted by the TryHeartMe shop interface.

1. **Account Creation:** The app requires authentication before giving access to the full checkout features. I registered a fresh user account and logged in.
    
2. **The Session Mechanism:** Inspecting how the application tracks state, it doesn't use standard session IDs. Instead, it relies on a client-side **JSON Web Token (JWT)** stored within the browser cookies.
    

### Phase 2: Intercepting and Analyzing the JWT

To view the token:

1. Open your browser's **Developer Tools** (`F12`).
    
2. Head over to **Application** (or **Storage**) -> **Cookies**.
    
3. Select the challenge domain to find a highly structured cookie string split by two periods (`.`), following the standard layout: `header.payload.signature`.
    

Copying this entire token string over to a decoder like **jwt.io** (or running `echo <token_part> | base64 -d` in the terminal) reveals the data structures inside:

- **Header:** Specifies the signing algorithm (e.g., `{"alg":"HS256","typ":"JWT"}`).
    
- **Payload:** Contains client data trusted by the application:
    
    JSON

    ```
    {
      "email": "user@mail.com",
      "role": "user",
      "credits": 0,
      "theme": "valentine"
    }
    ```

#### 2. The Forged Payload

Escalate the authorization level to `admin` and crank up the `credits` field:

JSON

```
{
  "email": "user@mail.com",
  "role": "admin",
  "credits": 10000,
  "theme": "valentine"
}
```

#### 3. Reassembling the Token

I base64-encoded (URL-safe) both pieces individually and joined them together with a period. Crucially, because the algorithm is `none`, **the third signature part is left blank**, but the trailing period remains:

Plaintext

```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJlbWFpbCI6InVzZXJAbWFpbC5jb20iLCJyb2xlIjoiYWRtaW4iLCJjcmVkaXRzIjoxMDAwMCwidGhlbWUiOiJ2YWxlbnRpbmUifQ.
```

### Phase 4: Privilege Escalation & Flag Recovery

1. **Injecting the Exploit:** I went back to the browser's Cookie Developer tools, replaced the original JWT token string with the newly forged one, and refreshed the page.
    
2. **Verifying the Bypass:** The application successfully accepted the signature-less token. The UI adjusted immediately, reflecting an Admin role status and a balance of 10,000 credits.
    
3. **Buying the Flag:** Navigating to the restricted administrative panel path at `http://<MACHINE_IP>:5000/admin`, a hidden store option appeared. Purchasing the restricted item triggered an application redirect to the checkout receipt path: `/receipt/valenflag`.
    

> **Flag Captured:** `THM{v4l3nt1n3_jwt_c00k13_t4mp3r_4dm1n_sh0p}`

## Vulnerability Remediation & Key Takeaways

- **Never Trust the Client:** Storing sensitive authorization claims (like `role` or wallet balances) purely client-side without strict server-side validation is highly dangerous.
    
- **Enforce Strict Algorithmic Restrictions:** Modern JWT backend libraries must explicitly disable or block the `none` algorithm during verification states (`JWT.verify()`).
    
- **Enforce Rigorous Signature Verification:** A JWT is only as secure as its signing mechanism. The application backend must actively reject any tokens that do not possess a signature verified against a robust, securely stored server secret key.
