# Lab — Enterprise Certificate Analysis: servicenow.com

## Overview

The goal of this lab was to analyze how a real enterprise company uses TLS certificates in production. I chose servicenow.com as my target and investigated its certificate fields, chain of trust, TLS configuration, and CA history. 

---

## Steps Performed

1. Connected to `servicenow.com` on port 443 using OpenSSL in Git Bash to capture the full certificate chain output
2. Extracted the leaf certificate from the chain output and inspected its fields — CN, SANs, issuer, validity dates, and certificate type
3. Used `curl -sI` to pull HTTP response headers and look for infrastructure clues like CDN or load balancer indicators
4. Ran the domain through SSL Labs to get a full TLS configuration grade and check for HSTS, protocol support, and OCSP Must-Staple
5. Searched crt.sh (Certificate Transparency logs) to view CA history, certificate count, and issuance patterns over time

---

## Results

**Certificate chain (OpenSSL):**
```
depth=2  C=US, O=Amazon, CN=Amazon Root CA 1
depth=1  C=US, O=Amazon, CN=Amazon RSA 2048 M04
depth=0  CN=servicenow.com
Verify return code: 0 (ok)
```

**Leaf certificate fields:**
- **CN:** servicenow.com
- **O:** (not present — DV certificate)
- **SANs:** DNS:servicenow.com, DNS:www.servicenow.com
- **Issuer:** C=US, O=Amazon, CN=Amazon RSA 2048 M04
- **Valid From:** May 21 00:00:00 2025 GMT
- **Valid Until:** Jun 19 23:59:59 2026 GMT
- **Days Remaining:** ~62 days (as of April 19, 2026)

**curl headers:**
```
Server: Apache
```
No CDN headers (no `cf-ray`, `x-cache`, `x-amz`) returned.

**SSL Labs results:**
- Grade: **A-**
- TLS 1.0 / 1.1: Disabled
- TLS 1.2: Enabled
- HSTS: Yes, long duration
- OCSP Must-Staple: No

**crt.sh (CT logs):**
- Total certificates issued: ~90 since 2013
- Active CAs: Entrust (2013–2024), Amazon ACM (2023–present, apex domain), DigiCert (2025–present, www subdomain)
- Typical validity: 12–13 months
- Notable SAN found in CT history: `prodwaf-www.servicenow.com`

---

## Key Findings

- The cert is issued by Amazon ACM, so TLS ends at an AWS load balancer, not the Apache server
- It's a DV cert — no company name in the certificate, just the domain
- Only 2 SANs, no wildcards — covers exactly `servicenow.com` and `www.servicenow.com`
- HSTS is enabled, so browsers always use HTTPS automatically
- CT logs show they switched from Entrust to Amazon ACM around 2023
- A `prodwaf-www.servicenow.com` SAN in CT history hints at a WAF sitting in front of the web server

---

## Explanation

The ACM issuer tells you where TLS ends — at the AWS load balancer, before traffic reaches Apache. That's normal for cloud deployments.

DV is fine at the edge because the cert just needs to encrypt the connection. HSTS makes sure browsers never try HTTP in the first place.

CT logs showed the company moved to ACM around 2023, which means they automated certificate renewal through AWS. The WAF subdomain showing up in older CT records is an example of how CT logs can leak infrastructure details unintentionally.

The clean verify return code means the chain is valid and any client would trust the certificate with no warnings.

---

## Challenges / Troubleshooting

- **PowerShell pipe issue:** Piping OpenSSL output in PowerShell corrupted the certificate data. Switched to Git Bash where the pipe works correctly with `echo Q | openssl s_client ... > file.txt`
- **Literal placeholder:** Initially ran the command with `[hostname]` typed literally instead of the actual domain name — corrected by replacing `[hostname]` with `servicenow.com`

---

## Artifacts

- `lab-01-enterprise-certificate-analysis.md` — this completed lab write-up
- `full_chain_output.txt` — raw OpenSSL s_client output containing the full certificate chain from servicenow.com
- `enterprise_cert.pem` — the extracted leaf certificate in PEM format

---

*CVI PKI Career Pathway — Foundations Phase*