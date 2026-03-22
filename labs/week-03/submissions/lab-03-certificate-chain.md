
# Lab 03 — Verify a Certificate Chain

## Overview
Briefly describe what this lab was about in your own words.
What PKI concept were you investigating?

---

## Environment
- OS:
- Terminal used (Mac Terminal / Git Bash / WSL):
- OpenSSL version (`openssl version`):
- Website used: github.com

---

## Chain Verification Result
Paste the output of your `openssl verify` command:

---

## Certificate Roles

| Certificate  | Role                        | Key Indicator                    |
|--------------|-----------------------------|----------------------------------|
| root.pem     |                             |                                  |
| intermediate.pem |                         |                                  |
| server.pem   |                             |                                  |

---

## Observations

1. Did the chain verify successfully? What did the output say?
2. How did you identify the root CA?
3. How did you identify the intermediate CA?
4. What field confirms whether a certificate can issue other certificates?
5. Why does removing the intermediate certificate break the chain?
