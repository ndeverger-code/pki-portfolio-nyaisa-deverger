# Lab 01 — Inspect X.509 Certificate Fields

## Overview
Briefly describe what this lab was about in your own words.
What PKI concept were you investigating?

The structure of X.509 certificates and the fields within the certificate

---

## Environment
- OS: Windows 
- Terminal used (Mac Terminal / Git Bash / WSL): PowerShell
- OpenSSL version: OpenSSL 3.5.5 27 

---

## Certificate Fields

| Field                | Value from your output |
|----------------------|------------------------|
| Version              |  3 (0x2)                      |
| Serial Number        |   aa:23:02:42:8e:f4:39:7e:10:bb:2c:32:93:1c:fc:2e                      |
| Signature Algorithm  |   ecdsa-with-SHA256                     |
| Issuer               |   C=US, O=Google Trust Services, CN=WE2                     |
| Subject              |  CN=*.google.com                      |
| Not Before           |  Feb 23 18:19:56 2026 GMT                      |
| Not After            |  May 18 18:19:55 2026 GMT                     |
| Public Key Algorithm |  id-ecPublicKey                      |

---

## Observations

1. Who issued the certificate? Google Trust Services
2. What domain or organization does it represent? CN=*.google.com 
3. When does it expire? May 18 18:19:55 2026 GMT 
4. What public key algorithm is used?  id-ecPublicKey 
5. Why does the Issuer field matter in a PKI system?
   
The issuer field is important because this establishes the chain of trust and tells the browser which CA signed the certificate.
The issuer field is also important in that is helps to prevent Man In The Middle Attacks. When the browser is checking the issuer, if the issuer is not trusted this will prevent a connection from happening and the trust chain is broken. 
