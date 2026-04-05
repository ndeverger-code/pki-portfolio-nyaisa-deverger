# Lab 01 — Convert Certificate Formats

## Overview
Briefly describe what this lab was about in your own words.
What PKI concept or system behavior were you investigating?
This lab focused on working with X.509 certificates in different formats using OpenSSL. The main PKI concepts explored were certificate encoding (PEM vs DER), certificate conversion, and bundling certificates and private keys into a PFX file.


---

## Environment
- Operating System: Windows 10
- Terminal Used: Powershell/OpenSSL
- OpenSSL Version (`openssl version`):OpenSSL 3.x

---

## Steps Performed
Summarize the key steps you performed to complete the lab.
Do not copy the lab instructions — describe what you actually did.

1. Retrieved the leaf certificate from google.com using OpenSSL and saved it as a PEM file.
2. Converted the PEM certificate to DER format.
3. Restored the PEM certificate from the DER file.
4. Generated a new RSA private key and created a self-signed test certificate.
5. Bundled the test certificate and private key into a PFX file.

---

## Results
Include the important outputs or findings from the lab.

- What did the PEM file look like compared to the DER file?
The PEM file was base64-encoded text with `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----` headers and footers.
- What happened when you opened the .der file in a text editor?
Opening the .der file in a text editor showed unreadable binary data, confirming it was not base64-encoded.
- What did the diff output show after converting PEM → DER → PEM?
After converting PEM → DER → PEM, the certificate content remained the same, but the formatting (line breaks, etc.) could differ; the actual certificate data was unchanged.
- What information was displayed when you verified the PFX?
Verifying the PFX displayed information about the certificate and private key, and prompted for the export password.

If you include screenshots, store them in the assets folder and reference them here:
![Description](../../assets/screenshots/week-04/your-screenshot.png)

---

## Key Findings
Document the most important observations from the lab.

• PEM is a human-readable, base64-encoded format with clear headers and footers.  
• DER is a binary format, not human-readable, but useful for certain applications.  
• PFX bundles both certificate and private key, and is protected by a password.

---

## Explanation
Explain why the results matter.

- Why does a PFX require a password?
A PFX requires a password to protect the private key and certificate bundle from unauthorized access.
- In what real-world scenario would you choose PEM vs DER vs PFX?
PEM is commonly used for web servers and text-based configurations; DER is used for binary protocols and some devices; PFX is used for securely transporting certificates and private keys together.
- Why is it important never to commit private key files to GitHub?
Private key files should never be committed to GitHub to prevent security breaches and unauthorized access to encrypted resources.
---

## Challenges / Troubleshooting
Document any issues encountered and how you resolved them.

- Initially, commands with backslashes (`\`) for line continuation did not work in Windows Command Prompt; resolved by entering commands on a single line.
- The DER file was missing at first due to a typo in the output path; corrected the path and re-ran the command.
- No output was shown when creating the PFX file, but the file was created successfully after confirming the command syntax.
---

## Artifacts
List the files generated during this lab.

- leaf_cert.pem
- leaf_cert.der
- leaf_cert_restored.pem
- test_cert.pem
- test_bundle.pfx