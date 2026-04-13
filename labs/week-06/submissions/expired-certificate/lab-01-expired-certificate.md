# Lab 01 — Expired Certificate

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## Incident Summary

**Target system:** portal.metrogeneral.org (simulated via expired.badssl.com)
**Diagnosed by:** Nyaisa Deverger
**Date of diagnosis:** April 12, 2026

---

### What failed

The TLS connection failed because the server's leaf certificate expired on April 12, 2015 — over 11 years ago.

---

### Evidence

- Not Before: Apr 9 00:00:00 2015 GMT / Not After: Apr 12 23:59:59 2015 GMT
- The certificate expired 4,018 days ago (over 11 years)
- Certificate chain is structurally complete — leaf → intermediate → root (3 certificates presented)
- Verify return code: 10 (certificate has expired)

**Supporting commands:**
```
openssl x509 -in expired_cert.pem -noout -dates
openssl s_client -connect expired.badssl.com:443 -showcerts </dev/null 2>/dev/null
```

---

### Why it failed

During the TLS handshake, the client checks the server certificate's validity period against the current date. Because the Not After date (April 12, 2015) has long passed, the client rejects the certificate with verify return code 10 — "certificate has expired." This is the same certificate lifecycle failure discussed in Week 5, Lesson 3: once a certificate passes its Not After date, it is no longer trusted regardless of whether the rest of the chain is valid.

---

### Chain status

The certificate chain is structurally intact. Three certificates were presented: the leaf (*.badssl.com) → intermediate (COMODO RSA Domain Validation Secure Server CA) → COMODO RSA Certification Authority (cross-signed by AddTrust External CA Root). No chain-related errors were found separate from the expiration failure.

---

### Remediation path

1. Generate a new private key and CSR for portal.metrogeneral.org (or use the existing key if it has not been compromised)
2. Submit the CSR to a trusted Certificate Authority for issuance
3. Receive the new leaf certificate and any required intermediate certificates from the CA
4. Install the new leaf certificate and intermediate chain on the server
5. Restart or reload the web server to apply the new certificate
6. Verify the new certificate is being served by connecting with `openssl s_client` and confirming the Not After date is in the future
7. Test from a browser to confirm no TLS errors

---

### Prevention

Implement automated certificate monitoring that alerts the operations team 30, 14, and 7 days before any certificate's Not After date, so renewals are initiated well before expiration.

---

## Diagnostic Steps

Document each step of the PKI Diagnostic Framework as you worked through it.

### Step 1 — Retrieve

**Command used:**

```
openssl s_client -connect expired.badssl.com:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM > expired_cert.pem
```

**What you observed:**

The command completed successfully (exit code 0) and produced expired_cert.pem. An SSL verification error related to the expired certificate is expected during the connection, but the certificate was retrieved successfully. The stderr output was suppressed with 2>/dev/null.

---

### Step 2 — Parse

**Command used:**

```
openssl x509 -in expired_cert.pem -text -noout
openssl x509 -in expired_cert.pem -noout -dates
```

**Key fields from the certificate:**

| Field | Value |
|---|---|
| Subject CN | *.badssl.com |
| Issuer | COMODO RSA Domain Validation Secure Server CA |
| Not Before | Apr  9 00:00:00 2015 GMT |
| Not After | Apr 12 23:59:59 2015 GMT |
| SAN entries | DNS:*.badssl.com, DNS:badssl.com |

**What you found:**

The certificate was only valid for 3 days (April 9–12, 2015) and expired over 11 years ago. It is well outside its validity period, confirming expiration as the root cause of the TLS failure.

---

### Step 3 — Validate the Chain

**Command used:**

```
openssl s_client -connect expired.badssl.com:443 -showcerts </dev/null 2>/dev/null
```

**Result:**

Chain valid — three certificates were presented in the chain. The only error reported was verify return code: 10 (certificate has expired).

**What you found:**

The chain is structurally complete: leaf (*.badssl.com) → intermediate (COMODO RSA Domain Validation Secure Server CA) → COMODO RSA Certification Authority. No chain-related issues were found. The expiration of the leaf certificate is the sole cause of the failure.

---

### Step 4 — Check Revocation and Trust

**Command used:**

```
openssl x509 -in expired_cert.pem -noout -text | grep -A1 "OCSP"
```

**What you found:**

An OCSP responder URL is present: http://ocsp.comodoca.com. However, revocation checking is not relevant to this remediation. An expired certificate is rejected by the client during validity period checking, which occurs before revocation status is consulted. The fix is to renew or replace the certificate, not to check or change its revocation status.

---

## Reflection

This lab stressed the importance of the certificate validity period in the TLS trust model. Even though the chain was structurally complete and no other errors existed, a single expired leaf certificate was enough to break the entire connection. It also clarified that expiration is checked before revocation — understanding the order of validation steps matters when diagnosing real incidents.

---

*CVI PKI Career Pathway — Foundations Phase*