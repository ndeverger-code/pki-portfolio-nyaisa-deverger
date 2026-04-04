# Lab 02 — Inspect Your Trust Store

## Overview
In this lab, I explored the trusted root certificate authorities (CAs) installed on my Windows system. The main PKI concept I investigated was how trust stores work and how certificates are validated using the list of trusted root CAs.

---

## Environment
- Operating System: Windows 10
- Terminal Used: VS Code Powershell
- OpenSSL Version (`openssl version`): Not applicable (used Windows tools)

---

## Steps Performed
Summarize the key steps you performed to complete the lab.
Do not copy the lab instructions — describe what you actually did.

1. Opened the Windows Certificate Manager using `certmgr.msc`.
2. Navigated to the "Trusted Root Certification Authorities" store and reviewed the list of installed root CAs.
3. Inspected the details of a selected root CA certificate, focusing on fields like Subject, Public Key, and Signature Algorithm.
4. Counted the total number of trusted root CAs and noted details for one specific CA.(35)

---

## Results
Include the important outputs or findings from the lab.

- I found approximately 35 trusted root CAs on my system.
- Example root CA inspected:
  - Subject: 'CN = DigiCert Global Root G2'
  - Expiration Date: Friday,January 15, 2038 8:00:00 AM
  - Public Key Algorithm: `rsa (2048 Bits)` (RSA)
  - Signature Algorithm: `sha1RSA`
- The verify return code output (from certificate details) indicated the certificate was trusted and valid.

If you include screenshots, store them in the assets folder and reference them here:
![Root CA Details](../../../../assets/screenshots/week-04/root-ca-details.png)
Broken Trust Example
---

## Key Findings
Document the most important observations from the lab.

• Windows comes with many pre-installed root CAs from various organizations.
• The Public Key Algorithm is found under the "Public Key" field, not always labeled directly.
• Signature Algorithm and Public Key Algorithm are related but distinct fields.

---

## Explanation
Explain why the results matter.

- Your browser trusts certificates from websites you have never visited before because their root CA is already in the system's trust store.
- If an enterprise's internal root CA was NOT in the trust store, certificates issued by it would not be trusted, causing errors for internal sites.
- I was surprised by how many root CAs are pre-installed, representing many different organizations.

---

## Challenges / Troubleshooting
Document any issues encountered and how you resolved them.

- It was initially confusing to find the Public Key Algorithm, as it is labeled "Public Key" and not always obvious. I resolved this by checking the value field for the algorithm name and bit length.
- When running openssl s_client, I initially received Verify return code: 20. 
This occurred because the standalone OpenSSL binary on Windows does not automatically reference the Windows 'Trusted Root Certification Authorities' 
store.
 This shows a key PKI concept: applications must be explicitly configured to point to a valid Trust Store to verify certificates."


---

## Artifacts
List the files generated during this lab.

- No PEM file generated (used Windows Certificate Manager)
- Screenshot stored in assets/screenshots/week-04/root-ca-details.png