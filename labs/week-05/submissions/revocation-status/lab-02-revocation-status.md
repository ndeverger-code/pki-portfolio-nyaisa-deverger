# Lab 02 — Check Certificate Revocation Status with OCSP

## Overview
This lab focused on checking the revocation status of a live TLS certificate using OCSP (Online Certificate Status Protocol). I connected to a live website, retrieved its leaf and issuer certificates, extracted revocation-related extensions (OCSP URL, CRL Distribution Point), queried the OCSP responder, and compared OCSP with CRL as two different approaches to certificate revocation checking.

## Environment
- Operating System: Windows 
- Terminal Used: Git Bash and PowerShell
- OpenSSL Version (openssl version): 
- Website Used for Certificate Retrieval: http://ocsp.sectigo.com (subject was Github.com)

## Steps Performed
1. Connected to a live website using `openssl s_client -connect` and extracted the leaf certificate, saving it as `leaf_cert.pem`.
2. Retrieved the full certificate chain from the TLS handshake and saved the issuer (intermediate CA) certificate as `issuer_cert.pem`.
3. Inspected the leaf certificate using `openssl x509 -text -noout` to locate the Authority Information Access (AIA) extension containing the OCSP responder URL and the CRL Distribution Point.
4. Extracted the OCSP URL (`http://ocsp.sectigo.com`) from the certificate and sent an OCSP query using both the leaf and issuer certificates with `openssl ocsp`.
5. Interpreted the OCSP response saved to `ocsp_response.txt`, confirming the certificate's revocation status, and documented the differences between OCSP and CRL.

## Results
- **What OCSP URL did you find in the certificate's Authority Information Access extension?**
  `http://ocsp.sectigo.com` — This was listed under the Authority Information Access (AIA) extension of the leaf certificate.

- **What was the OCSP response status (`good`, `revoked`, or `unknown`) and what does it mean?**
  The response status was **`good`**. This means the certificate is currently valid and has **not** been revoked by the issuing CA. It is safe to trust for the purposes of establishing a TLS connection.

- **What were the `This Update` and `Next Update` values in the OCSP response, and what do they indicate?**
This Update: Apr  6 00:00:00 2026 GMT
Next Update: Apr 13 00:00:00 2026 GMT

`This Update` indicates the timestamp when the OCSP responder generated the response. `Next Update` indicates when the response expires and a new query should be made. 

- **Where was the CRL Distribution Point located in the certificate?**
  The CA Issuers URI found in the AIA extension was: `http://crt.sectigo.com/SectigoPublicServerAuthenticationCADVE36.crt`. The CRL Distribution Point URL is listed separately under the **X509v3 CRL Distribution Points** extension in the certificate.

## Key Findings
- **Subject and Issuer of the leaf certificate:** The Subject identifies the specific domain the certificate was issued to. The Issuer identifies the intermediate CA — **Sectigo Public Server Authentication CA DV E36** — that signed the leaf certificate.
- **Why the Subject and Issuer of `issuer_cert.pem` differ from the leaf cert:** The issuer certificate is an **intermediate CA certificate**. It has its own Subject (the intermediate CA's name) and its own Issuer (the root CA that signed it). This is what forms the **chain of trust**: Leaf → Intermediate CA → Root CA. Each certificate in the chain is signed by the entity above it, and the root CA is self-signed and pre-trusted by operating systems and browsers.
- The OCSP responder URL (`http://ocsp.sectigo.com`) is what the certificate tells clients to query for real-time revocation status.
- The certificate's revocation status was confirmed as `good`, meaning it has not been revoked.

## Explanation
- **What is the difference between OCSP and CRL as revocation checking methods?**
  OCSP provides a **real-time, per-certificate** status check — the client sends a request for one specific certificate and gets back a small response (`good`, `revoked`, or `unknown`). CRL is a **bulk list** of all revoked certificate serial numbers published by the CA on a regular schedule. For **high-traffic systems, OCSP is better** because the response is small and targeted, whereas a CRL can grow very large as more certificates are revoked, requiring clients to download and parse the entire list. 

- **Why does an OCSP query require both the leaf certificate and the issuer certificate?**
  The OCSP request must identify the specific certificate being checked. It does this by including the **issuer's name hash**, the **issuer's public key hash**, and the **leaf certificate's serial number**. Without the issuer certificate, the client cannot compute the issuer name and key hashes, and the OCSP responder cannot determine which certificate is being asked about.

- **In what scenario would a certificate show `unknown` status from an OCSP responder?**
  A certificate may return `unknown` if the OCSP responder **has no record** of it — for example, if the certificate was not issued by the CA that the responder serves, if the responder's database is out of sync, or if the certificate's serial number is not recognized. 

## Challenges / Troubleshooting
- Bash-specific commands (`grep`, `sed`, `tr`, `$(...)`) do not work in PowerShell on Windows. I switched to **Git Bash** to run the OCSP URL extraction commands successfully.
- In Git Bash, Windows paths must use the format `/c/Users/...` with forward slashes instead of `C:\Users\...`.
- Had to abort a pending `git rebase` operation using `git rebase --abort` before Git Bash would function normally.

## Artifacts
- leaf_cert.pem, issuer_cert.pem, ocsp_response.txt