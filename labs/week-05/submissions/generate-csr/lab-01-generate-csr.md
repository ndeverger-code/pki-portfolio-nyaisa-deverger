# Lab 01 — Generate a CSR and Simulate the Issuance Workflow

## Overview
In this lab, I practiced generating a Certificate Signing Request (CSR) and simulating the process of certificate issuance and validation. I investigated how a CSR is created, how a certificate is self-signed, and how to verify that the certificate matches the original private key. The main PKI concepts explored were the CSR workflow, X.509 certificate fields, and the relationship between private keys and certificates.

---

## Environment
- Operating System: Windows 10
- Terminal Used: PowerShell (VS Code integrated terminal)
- OpenSSL Version (`openssl version`):

---

## Steps Performed
Summarize the key steps you performed to complete the lab.
Do not copy the lab instructions — describe what you actually did.

1. Generated a 2048-bit RSA private key and saved it as `test_key.pem`.
2. Created a CSR (`test_csr.pem`) using the private key and specified subject fields.
3. Inspected the CSR to verify the subject fields and public key.
4. Self-signed the CSR to produce a certificate (`test_cert.pem`) valid for 365 days.
5. Extracted and compared the public key from both the private key and the certificate to confirm they matched.

---

## Results
Include the important outputs or findings from the lab.

- **Subject fields in the CSR:**
  - Common Name (CN): lab01.cvi.internal
  - Organization (O): CyberVisionaries Institute
  - Organizational Unit (OU): PKI Career Pathway
  - Country (C): US
  - State (ST): California
  - Locality (L): San Francisco
  - Each field identifies the entity requesting the certificate and provides organizational and location information.
- **Issuer field in self-signed certificate:**  
  The Issuer is identical to the Subject because the certificate was self-signed (signed with its own private key).
- **Public key comparison:**  
  The comparison showed no differences, confirming the certificate contains the public key corresponding to the private key.
- **X.509 fields:**  
  The fields in the signed certificate matched what was learned about X.509 structure, including Subject, Issuer, Validity, and Public Key.

If you include screenshots, store them in the assets folder and reference them here:
![CSR and Certificate Details](../../assets/screenshots/week-05/your-screenshot.png)

---

## Key Findings
Document the most important observations from the lab.

• The CSR contains all the subject information and the public key, but not the private key.  
• A self-signed certificate has identical Subject and Issuer fields.  
• Matching public keys confirm the certificate was generated from the correct private key.

---

## Explanation
Explain why the results matter.

- The CSR is used to request a certificate from a CA without exposing the private key, maintaining security.
- The CA receives only the CSR (public key and subject info), never the private key, to prevent compromise.
- If the public key in the certificate did not match the private key, the certificate would not work for authentication or encryption, breaking trust.

---

## Challenges / Troubleshooting
Document any issues encountered and how you resolved them.

- Accidentally used Unix-style line continuations (`\`) in PowerShell, which caused errors. Fixed by entering commands on a single line.
- The `head` and `diff` commands were not available in PowerShell; used `Select-Object -First 5` and `fc.exe` instead.

---

## Artifacts
List the files generated during this lab.

- test_key.pem - NOT pushed up to GitHub
- test_csr.pem
- test_cert.pem
- cert_pubkey.pem - Not requested in submission
- key_pubkey.pem - Not requested in submission
