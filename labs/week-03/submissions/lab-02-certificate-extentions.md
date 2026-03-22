
# Lab 02 — Investigate Certificate Extensions

## Overview
Briefly describe what this lab was about in your own words.
What PKI concept were you investigating?

Certificate Extentions in a Leaf Certificate - Inspection of SANS/Key Usage and EKU

---

## Environment
- OS: Windows
- Terminal used (Mac Terminal / Git Bash / WSL): Powershell
- OpenSSL version 

---

## Extensions Found

### Subject Alternative Name (SAN)
Paste the value from your output: DNS:*.google.com

### Key Usage
Paste the value from your output: Critical Digital Signature

### Extended Key Usage (EKU)
Paste the value from your output: TLS Web Server Authentication

### Basic Constraints
Paste the value from your output: critical CA:FALSE

---

## Observations

1. What domains appear in the SAN field? *.google.com
2. What is this certificate authorized to do based on Key Usage? It is authorized to provide a Digital Signature
3. What does the EKU field tell you about this certificate's purpose? That the EKU is authorized to sign a web server to a client 
4. Is this a CA certificate? How can you tell? No - X509v3 Basic Constraints: critical
CA:FALSE if this were a CA certificate this would read as true
5. Why does SAN matter more than the Subject CN field in modern TLS? Previously, CN was used however it could only hold one entity whereas the
   SAN will hold multiple hosts domains subdomains etc. 
