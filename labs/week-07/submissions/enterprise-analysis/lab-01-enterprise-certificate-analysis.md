# Lab — Enterprise Certificate Analysis: servicenow.com

## Overview

The goal of this lab was to analyze how a real enterprise company uses TLS certificates in production. I chose servicenow.com as my target and investigated its certificate fields, chain of trust, TLS configuration, and CA history.

---

## 1. Certificate Summary

**Target domain:** servicenow.com  
**Certificate type:** Domain Validated (DV)  
**Evidence for DV:** The `O` (Organization) field is completely missing from the certificate. DV certificates only verify that you own the domain — they do not check who the company is — so there is no company name anywhere in the Subject. That missing `O` field is how you know it is a DV cert.

**Leaf certificate fields:**
| Field | Value |
|---|---|
| CN | servicenow.com |
| O | (not present — DV) |
| SANs | DNS:servicenow.com, DNS:www.servicenow.com |
| Issuer | C=US, O=Amazon, CN=Amazon RSA 2048 M04 |
| Valid From | May 21 00:00:00 2025 GMT |
| Valid Until | Jun 19 23:59:59 2026 GMT |
| Days Remaining | ~62 days (as of April 19, 2026) |
| Key Algorithm | RSA 2048 |

**Certificate chain (OpenSSL):**
```
depth=2  C=US, O=Amazon, CN=Amazon Root CA 1
depth=1  C=US, O=Amazon, CN=Amazon RSA 2048 M04
depth=0  CN=servicenow.com
Verify return code: 0 (ok)
```

---

## 2. Chain Analysis

The certificate chain has three levels:

- **Root CA:** Amazon Root CA 1 — this is the top-level certificate that browsers and operating systems already trust
- **Intermediate CA:** Amazon RSA 2048 M04 — sits in the middle and was used to sign the leaf certificate
- **Leaf:** servicenow.com — the actual certificate the server shows to visitors

The `Verify return code: 0 (ok)` means the whole chain checked out — each certificate was properly signed by the one above it, and the root is trusted. No errors, no missing pieces.

---

## 3. Termination Analysis

**curl header result:**
```
Server: Apache
```
No CDN-specific headers present (`cf-ray`, `x-cache`, `x-amz-cf-id`).

**Conclusion: TLS terminates at the AWS load balancer, not at Apache.**

The certificate is issued by Amazon ACM (AWS Certificate Manager), which only works with AWS load balancers — ACM does not let you export the private key to put on your own server. That means the encrypted connection ends at the load balancer, which then forwards the unencrypted traffic to the Apache server sitting behind it. The `Server: Apache` header tells you Apache is back there, but Apache never actually handled the TLS part. This is a very common setup for large websites running on AWS.

---

## 4. TLS Configuration

**SSL Labs grade: A-**

| Protocol | Status |
|---|---|
| TLS 1.3 | Yes — supported |
| TLS 1.2 | Yes — enabled |
| TLS 1.1 | No — disabled |
| TLS 1.0 | No — disabled |

**Additional settings:**
- **HSTS:** Yes — long-duration `max-age` set; browsers enforce HTTPS-only automatically
- **OCSP Stapling:** No — the server does not staple OCSP responses; clients must perform live OCSP lookups to check revocation status
- **OCSP Must-Staple extension:** Not set in the certificate

**A- grade explanation:** The A- instead of A+ comes down to HSTS not being on the browser preload list. HSTS already protects people who have visited before, but someone visiting for the very first time on a new browser does not get that protection until after their first connection. Getting added to the preload list would fix that and push the grade to A+.

---

## 5. CT Log Analysis

**Source:** crt.sh (Certificate Transparency log aggregator)

| Finding | Detail |
|---|---|
| Total certificates issued | ~90 since 2013 |
| Typical validity period | 12–13 months |
| CA from 2013–2022 | Entrust |
| CA from 2023–present (apex) | Amazon ACM |
| CA from 2025–present (www) | DigiCert |
| Notable historical SAN | `prodwaf-www.servicenow.com` |

**CA migration:** CT logs show that ServiceNow used Entrust from 2013 up through around 2022, then switched to Amazon ACM starting in 2023. Moving to ACM means certificate renewals happen automatically through AWS — no one has to manually track when certs expire and remember to replace them.

**Infrastructure leakage via SAN:** One of the older certificates in CT logs had a SAN called `prodwaf-www.servicenow.com`. That name gives away that there is a Web Application Firewall (WAF) sitting in front of the web server. ServiceNow never publicly announced that, but because every certificate gets logged in CT, that internal hostname became publicly visible. This is actually one of the main reasons security teams watch CT logs — details like this show up even when a company does not intend to share them.

---

## 6. Architecture Assessment

ServiceNow handles its certificates through AWS, which means renewals happen automatically and there is no risk of someone forgetting to renew a cert and taking down the site. TLS ends at the load balancer before requests ever reach the Apache server behind it, which is a clean and common setup for large cloud-hosted applications. The WAF layer showing up in old CT log records tells you there is an extra security filter in front of the web server — ServiceNow did not advertise that, but CT logs made it visible anyway. The A- grade from SSL Labs shows the setup works well and is properly secured, but they have not done every possible hardening step, like HSTS preloading or OCSP stapling — which is a reasonable choice when you are managing certificates at this scale and want to keep things simple.

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