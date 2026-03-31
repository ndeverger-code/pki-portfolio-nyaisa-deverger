
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

*.google.com
*.google.ca
*.google.cl
*.google.co.in
*.google.co.jp
*.google.co.uk
*.google.com.ar
*.google.com.au
*.google.com.br
*.google.com.mx
*.google.com.tr
*.google.com.vn
*.google.de
*.google.es
*.google.fr
*.google.hu
*.google.it
*.google.nl
*.google.pl
*.google.pt
*.googleadservices.com
*.googlesyndication.com
*.googletagmanager.com
*.googletagservices.com
*.google-analytics.com
*.googletraveladservices.com
*.cloud.google.com
*.appengine.google.com
*.datacompute.google.com
*.googleapis.com
*.youtube.com
*.youtube-nocookie.com
*.ytimg.com
*.yt.be
*.youtubeeducation.com
*.youtubekids.com
*.gstatic.cn
*.gstatic-cn.com
*.gvt1.com
*.gvt2.com
googlecnapps.cn
*.googlecnapps.cn
*.googleapps.cn
*.googlecommerce.com
*.urchin.com
*.bdn.dev
*.origin-test.bdn.dev
*.googlezip.net
*.crowdsource.google.com

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
3. What does the EKU field tell you about this certificate's purpose? This certificate can be used to secure websites and also verify identities on both sides of a connection
4. Is this a CA certificate? How can you tell? No - X509v3 Basic Constraints: critical
CA:FALSE if this were a CA certificate this would read as true
5. Why does SAN matter more than the Subject CN field in modern TLS? Previously, CN was used however it could only hold one entity whereas the
   SAN will hold multiple hosts domains subdomains etc. 
