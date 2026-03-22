# Lab 04 — Detect Certificate Misconfigurations

## Overview

This lab focused on identifying and understanding common Public Key Infrastructure (PKI) certificate misconfigurations and how they impact browser trust and secure communication.

The primary PKI concepts investigated included:
- Subject Alternative Names (SAN)
- Extended Key Usage (EKU)
- Certificate expiration
- Certificate chain validation

---

## Scenario 1 — Missing Subject Alternative Name (SAN)

**Would modern browsers trust this certificate?**  
No. Modern browsers would flag this certificate as **Not Secure**.

### Analysis

Previously, browsers relied on the **Common Name (CN)** field to verify a website’s identity. However, this method is now deprecated. Modern security standards require the **Subject Alternative Name (SAN)** extension, which explicitly lists the domain names or IP addresses the certificate is valid for.

Without a SAN entry, the browser cannot confirm that the certificate matches the requested domain, even if the CN appears correct. As a result, users typically encounter errors such as:

- `SSL_ERROR_BAD_CERT_DOMAIN`
- “Your connection is not private”

---

## Scenario 2 — Incorrect Extended Key Usage (EKU)

**Would a browser accept this certificate for a web server?**  
No. The browser would reject it, even if all other certificate fields are valid.

### Analysis

The **Extended Key Usage (EKU)** field defines what a certificate is permitted to be used for. For HTTPS web servers, the certificate must include the **Server Authentication** EKU.

If the EKU is missing or set to a different purpose (such as client authentication or code signing), the browser will refuse to establish a secure connection. In this scenario, users would typically see an error such as:

- `ERR_CERT_INVALID`

---

## Scenario 3 — Expired Certificate

**What happens if this certificate is used today?**  
The browser will block access to the site and display a security warning.

### Analysis

Certificates are only valid within a defined time range specified by the **Not Before** and **Not After** fields. If the current date falls outside this range, the certificate fails validation.

This failure highlights the importance of proper certificate lifecycle management. When users attempt to access a site with an expired certificate, they commonly see errors such as:

- `ERR_CERT_DATE_INVALID`

---

## Scenario 4 — Missing Intermediate Certificate

**Can the browser build a complete trust chain?**  
No.

### Analysis

For a certificate to be trusted, the browser must be able to build a complete **chain of trust** from the server’s certificate (leaf certificate) through any intermediate certificates up to a trusted **Root Certificate Authority (CA)**.

If the server fails to provide the required intermediate certificate, the chain is incomplete. As a result, the browser cannot verify who issued the certificate and will reject it.

This issue is typically resolved by properly configuring the web server to serve the full certificate chain (often by combining certificates into a single chain file). Common errors include:

- `SEC_ERROR_UNKNOWN_ISSUER`

---

## Key Takeaway

The most important lesson from this lab is the critical role that **Extended Key Usage (EKU)** and **intermediate certificates** play in certificate trust validation. These elements act as silent gatekeepers—ensuring certificates are used for their intended purpose and that the chain of trust remains intact from the server to a trusted root authority.
