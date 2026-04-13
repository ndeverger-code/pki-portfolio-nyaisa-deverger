# Lab 02 — Broken Certificate Chain

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## Incident Summary

**Target system:** Radiology imaging platform (simulated via incomplete-chain.badssl.com)
**Diagnosed by:** Nyaisa Deverger
**Date of diagnosis:** April 12, 2026

---

### What failed

The TLS connection failed because the server only sent the leaf certificate without the required intermediate CA certificate, preventing the client from building a chain to the trusted root.

---

### Evidence

- Leaf certificate Subject: CN=*.badssl.com
- Issuer CN: R13 (C=US, O=Let's Encrypt, CN=R13) — this is the missing intermediate
- `openssl verify leaf_cert.pem` produced: error 20 at 0 depth lookup: unable to get local issuer certificate
- `openssl verify -untrusted issuer_cert.pem leaf_cert.pem` returned: leaf_cert.pem: OK
- Verify return code from s_client: 21 (unable to verify the first certificate)

**Supporting commands:**
```
openssl s_client -connect incomplete-chain.badssl.com:443 </dev/null 2>&1
openssl verify leaf_cert.pem
openssl verify -untrusted issuer_cert.pem leaf_cert.pem
```

---

### Why it failed

This was a server configuration problem, not a certificate problem. The leaf certificate itself is valid — it has correct dates, a matching subject, and proper SANs. But the server was not configured to send the intermediate CA certificate (R13) alongside the leaf. Without the intermediate, the client cannot build the trust path from the leaf to the root CA (ISRG Root X1), so it rejects the connection. This distinction matters because replacing the certificate would not fix the issue — the server configuration must be updated to include the full chain.

---

### Chain status

The chain is broken. The server sent only 1 certificate (the leaf). The intermediate CA (Let's Encrypt R13) was not included. After manually downloading the intermediate from http://r13.i.lencr.org/ and providing it to openssl verify with -untrusted, the chain validated successfully — confirming the root CA is trusted and the only issue is the missing intermediate in the server's configuration.

---

### Remediation path

1. Download the intermediate CA certificate from the CA Issuers URI (http://r13.i.lencr.org/) or Let's Encrypt's public repository
2. Convert from DER to PEM format if necessary: `openssl x509 -inform DER -in intermediate.der -out issuer_cert.pem`
3. Concatenate the leaf certificate and intermediate into a full chain file: `cat leaf_cert.pem issuer_cert.pem > fullchain.pem`
4. Update the server's TLS configuration to use the full chain file instead of the leaf-only file
5. Restart or reload the web server to apply the new configuration
6. Verify the fix by connecting with `openssl s_client -connect <host>:443 -showcerts` and confirming both leaf and intermediate are now sent
7. Confirm verify return code is 0 (ok)

---

### Prevention

Include a post-renewal verification step in the certificate deployment process that runs `openssl s_client -showcerts` against the server and confirms the full chain (leaf + intermediate) is being served before closing the maintenance window.

---

## Diagnostic Steps

Document each step of the PKI Diagnostic Framework as you worked through it.

### Step 1 — Retrieve

**Command used:**

```
openssl s_client -connect incomplete-chain.badssl.com:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM > leaf_cert.pem
openssl s_client -connect incomplete-chain.badssl.com:443 </dev/null 2>&1
```

**What you observed:**

The server sent only 1 certificate in the chain — the leaf (CN=*.badssl.com). No intermediate was included. The verify return code was 21: unable to verify the first certificate. Error 20 (unable to get local issuer certificate) was also reported.

---

### Step 2 — Parse

**Command used:**

```
openssl x509 -in leaf_cert.pem -text -noout
openssl x509 -in leaf_cert.pem -noout -text | grep -A3 "Authority Information Access"
```

**Key fields from the certificate:**

| Field | Value |
|---|---|
| Subject CN | *.badssl.com |
| Issuer | C=US, O=Let's Encrypt, CN=R13 |
| Not Before | Mar 24 20:02:52 2026 GMT |
| Not After | Jun 22 20:02:51 2026 GMT |
| SAN entries | DNS:*.badssl.com, DNS:badssl.com |

**What you found:**

The leaf certificate itself is valid — it is within its validity period, the subject and SANs are correct. The Issuer CN is R13 (Let's Encrypt), which identifies the missing intermediate. The Authority Information Access extension contains CA Issuers - URI:http://r13.i.lencr.org/, which points to where the intermediate can be downloaded.

---

### Step 3 — Validate the Chain

**Command used:**

```
openssl verify leaf_cert.pem
curl -s "http://r13.i.lencr.org/" -o intermediate.der
openssl x509 -inform DER -in intermediate.der -out issuer_cert.pem
openssl verify -untrusted issuer_cert.pem leaf_cert.pem
```

**Result:**

Chain broken — `openssl verify leaf_cert.pem` returned error 20: unable to get local issuer certificate. After downloading the intermediate from the CA Issuers URI, converting from DER to PEM, and re-running with -untrusted, verification succeeded: leaf_cert.pem: OK.

**What you found:**

The chain failure is caused entirely by the missing intermediate. The leaf certificate and root CA are both fine. This confirmed it is a server configuration issue — the intermediate was never installed alongside the leaf.

---

### Step 4 — Check Revocation and Trust

**Command used:**

```
openssl s_client -connect incomplete-chain.badssl.com:443 -showcerts </dev/null 2>/dev/null | grep "Verify return code"
```

**What you found:**

The server still returns verify return code 21 (unable to verify the first certificate) because it has not been fixed — it still does not send the intermediate. However, the local verification with -untrusted proved the root CA (ISRG Root X1) is trusted by the system. There is no indication of a revocation issue. This is purely a chain configuration problem.

---

## Reflection

This lab showed me that a perfectly valid certificate can still cause a TLS failure if the server isn't configured to send the full chain. The certificate wasn't expired, revoked, or misnamed — the problem was entirely in how the server was set up. The CA Issuers URI in the Authority Information Access extension was key to finding and downloading the missing intermediate, which is something I first saw in Week 3's certificate extension lab.

---

*CVI PKI Career Pathway — Foundations Phase*