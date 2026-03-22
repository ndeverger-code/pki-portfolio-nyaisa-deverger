
# Lab 03 — Verify a Certificate Chain

## Overview
Briefly describe what this lab was about in your own words.
What PKI concept were you investigating?

The purpose of this lab is to inspect and verify a certificate chain
---

## Environment
- OS: Windows
- Terminal used (Mac Terminal / Git Bash / WSL): PowerShell
- OpenSSL version (`openssl version`):
- Website used: github.com

---

## Chain Verification Result
Paste the output of your `openssl verify` command: server.pem: OK

---
## Certificate Roles

| Certificate | Role | Key Indicator |
| :--- | :--- | :--- |
| **root.pem** | Trust Anchor | Subject and Issuer are identical; located in the root store. |
| **intermediate.pem** | Issuing Certificate Authority | Subject matches server's Issuer; signs the leaf certificate. |
| **server.pem** | End Entity | `CA: FALSE` - It cannot issue other certificates. |

---

## Observations

1. Did the chain verify successfully? What did the output say? Initially No - I recieved an error :
error server.pem: verification failed - I then downloaded the Mozilla CA Bundle and this corrected the issue. I then was able to verify successfully   
3. How did you identify the root CA? The subject and the issuer field are the same
4. How did you identify the intermediate CA? I was able to identify because this is placed in the middle. The subject on the intermediate matches the issuer of the leaf certificate
5. What field confirms whether a certificate can issue other certificates? Basic Constraints
6. Why does removing the intermediate certificate break the chain? It will fail to verify because it breaks the trust chain
