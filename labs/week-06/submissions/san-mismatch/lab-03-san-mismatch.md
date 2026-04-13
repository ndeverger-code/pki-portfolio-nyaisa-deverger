# Lab 03 — Hostname and SAN Mismatch Diagnosis

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## Incident Summary

**Target system:** Staff scheduling portal (simulated via wrong.host.badssl.com)
**Diagnosed by:** Nyaisa Deverger
**Date of diagnosis:** April 12, 2026

---

### What failed

The server's TLS certificate does not list `wrong.host.badssl.com` (representing `staff.metrogeneral.org` in the scenario) as an authorized hostname in its Subject Alternative Name (SAN) extension. As a result, any standards-compliant browser or TLS client will reject the connection due to a hostname mismatch.

---

### Evidence

- **Hostname accessed:** `wrong.host.badssl.com`
- **Subject CN:** `*.badssl.com`
- **SAN entries:** `DNS:*.badssl.com`, `DNS:badssl.com`
- **The mismatch:** Per RFC 6125, the wildcard `*.badssl.com` covers only a single subdomain level (e.g., `www.badssl.com` or `api.badssl.com`). Because `wrong.host.badssl.com` contains two subdomain levels, it falls outside the scope of the wildcard. The literal entry `badssl.com` is also not a match.
- **Verify return code:** `0 (ok)` — the certificate chain validated successfully, confirming the failure is isolated to hostname validation

---

### Why it failed

The certificate and chain are valid, but the hostname isn’t listed in the SAN. Wildcards like *.badssl.com only match one subdomain, so wrong.host.badssl.com isn’t covered. Browsers check the SAN against the address you visit, and if there’s no match, they block the site. s_client doesn’t check hostnames, so it looks fine there, but a browser would fail.

---

### Chain status

The certificate chain is intact. The server presented two certificates: the leaf (`CN=*.badssl.com`) issued by `Let's Encrypt R13`, and the R13 intermediate itself, which chains to the trusted root `ISRG Root X1`. The `s_client` handshake returned `Verify return code: 0 (ok)`, confirming no chain or trust store issues. The failure is entirely attributable to the SAN mismatch.

---

### Remediation path

1. **Generate a new CSR** including `staff.metrogeneral.org` as a Subject Alternative Name. This can be accomplished with OpenSSL's `-addext "subjectAltName=DNS:staff.metrogeneral.org"` flag or through a configuration file specifying `[alt_names]` entries.
2. **Submit the CSR to the organization's CA** and request issuance of a certificate covering the new hostname.
3. **Deploy the new certificate** on the web server along with the complete chain (leaf and intermediate).
4. **Validate the deployment** by running `openssl s_client -connect staff.metrogeneral.org:443 -servername staff.metrogeneral.org` and confirming the SAN includes the expected hostname.
5. **Consider consolidating under a multi-domain SAN certificate** that authorizes both `scheduling.metrogeneral.org` and `staff.metrogeneral.org`, reducing the number of certificates to manage.

#### Why a DNS CNAME alias would not fix this

Adding a CNAME only changes where DNS sends the request, not what name the browser checks. The browser still expects the cert to list the hostname you typed. If staff.metrogeneral.org isn’t in the SAN, the connection fails, even if DNS points to a server with a valid cert for another name. The fix is to get a cert that lists the right hostname.

---

### Prevention

Implement a hostname-to-certificate mapping process. Before any new subdomain or service endpoint goes live, the deployment checklist should include verification that the hostname is covered by an existing certificate's SAN or that a new certificate has been requested. Automated certificate monitoring tools can also flag mismatches proactively before users encounter them.

---

## Diagnostic Steps

Document each step of the PKI Diagnostic Framework as you worked through it.

### Step 1 — Retrieve

**Command used:**

```
openssl s_client -connect wrong.host.badssl.com:443 -servername wrong.host.badssl.com </dev/null 2>&1
```

**What you observed:**

The TLS handshake completed successfully, and the certificate was retrieved without errors. The output showed `Verify return code: 0 (ok)`, indicating the chain validated properly. However, it is worth noting that `s_client` does not perform hostname-to-SAN matching — it only evaluates chain integrity. A browser performing the same connection would reject it due to the SAN mismatch, even though `s_client` reported success.

---

### Step 2 — Parse

**Command used:**

```
openssl x509 -in mismatch_cert.pem -text -noout
openssl x509 -in mismatch_cert.pem -noout -text | grep -A5 "Subject Alternative Name"
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

The SAN extension lists only two entries: `*.badssl.com` and `badssl.com`. Since `wrong.host.badssl.com` is a multi-level subdomain, it is not covered by the wildcard. It is also important to note that modern TLS implementations prioritize the SAN over the Subject CN for hostname validation — if a SAN extension is present, the CN is effectively ignored. Neither SAN entry authorizes the hostname in question.

---

### Step 3 — Validate the Chain

**Command used:**

```
openssl verify mismatch_cert.pem
```

**Result:**

Local verification failed: `error 20 at 0 depth lookup: unable to get local issuer certificate`. This occurred because `mismatch_cert.pem` contains only the leaf certificate — the intermediate was not bundled into the saved file.

**What you found:**

This error does not indicate a server-side chain problem. The `s_client` output from Step 1 confirmed that the server delivered a complete chain (leaf + R13 intermediate) and that verification succeeded with `Verify return code: 0 (ok)`. The local `openssl verify` failure is an artifact of how the certificate was saved, not a reflection of the deployment. This step effectively ruled out chain and trust issues, further isolating the root cause to the hostname/SAN mismatch.

---

### Step 4 — Check Revocation and Trust

**Command used:**

```
openssl x509 -in mismatch_cert.pem -noout -text | grep -A2 "OCSP"
```

**What you found:**

The grep returned no results, confirming the absence of an OCSP responder URL. This is consistent with Let's Encrypt's decision to discontinue OCSP in favor of CRL-based revocation. Regardless, revocation is not a factor in this incident — the certificate has not been revoked, and the failure mechanism is entirely unrelated to revocation status. The root CA (ISRG Root X1) is present in the system trust store, as confirmed by the successful chain validation in Step 1. The diagnosis is conclusive: the failure is attributable solely to the hostname not appearing in the certificate's SAN.

---

## Reflection

This lab reinforced an important distinction between chain validation and hostname validation — two checks that are often conflated but operate independently. The fact that `s_client` returned `Verify return code: 0 (ok)` while a browser would reject the same connection highlights that chain integrity and hostname authorization are separate layers of the TLS trust model. I found the CNAME discussion particularly valuable because it required thinking through the interaction between DNS resolution and TLS validation. The intuition that a CNAME should work is understandable, but once you trace the flow — the browser validates against the URL bar hostname, not the resolved target — it becomes clear why DNS-level redirects cannot substitute for proper SAN coverage. Understanding that separation is something I think will be relevant well beyond this lab.

---

*CVI PKI Career Pathway — Foundations Phase*