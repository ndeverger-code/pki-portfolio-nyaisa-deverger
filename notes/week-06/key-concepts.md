
# Week 6 Lesson Notes — PKI Incident Diagnosis & Troubleshooting

## 1. Core Concepts

This week focused on diagnosing why TLS connections fail, using a structured, step-by-step approach. Across four labs, I learned that certificate failures can be caused by expiration, broken chains, hostname/SAN mismatches, or trust store misconfigurations. The diagnostic framework is: retrieve the certificate, parse its details, validate the chain, and check revocation/trust. Each lab illustrated a different real-world failure scenario and how to resolve it.

---

## 2. Why It Matters

PKI failures are a leading cause of outages for both public websites and internal systems. Expired certificates, missing intermediates, or misconfigured hostnames can instantly break secure access for users and services. Diagnosing and fixing these issues quickly is critical for uptime and security. The labs showed how even small missteps (like forgetting to update a trust store or missing a SAN entry) can have major operational impacts.

---

## 3. Technical Breakdown (by Lab)

### Lab 1: Expired Certificate
- **What failed:** The server’s certificate was expired (Not After date in 2015), so all clients rejected the connection immediately.
- **Chain status:** Structurally complete (leaf → intermediate → root), but expiration overrides all else.
- **Key command:** `openssl x509 -in cert.pem -noout -dates`
- **Lesson:** Expiration is always checked first; nothing else matters if the cert is expired.

### Lab 2: Broken Chain
- **What failed:** The server only sent the leaf certificate, omitting the required intermediate CA. Clients could not build a chain to a trusted root.
- **Chain status:** Broken (missing intermediate). Manually adding the intermediate allowed validation.
- **Key command:** `openssl verify -untrusted issuer_cert.pem leaf_cert.pem`
- **Lesson:** Server misconfiguration (not certificate content) was the root cause. Always check what the server is actually sending.

### Lab 3: SAN/Hostname Mismatch
- **What failed:** The certificate’s Subject Alternative Name (SAN) did not include the hostname being accessed. Wildcards only match one subdomain level.
- **Chain status:** Intact and trusted, but hostname validation failed.
- **Key command:** `openssl x509 -in cert.pem -text | grep -A5 "Subject Alternative Name"`
- **Lesson:** Even with a valid chain, the connection fails if the hostname isn’t listed in the SAN. DNS aliases (CNAMEs) don’t fix this.

### Lab 4: Full Diagnostic Scenario
- **What failed:** Devices on a new subnet could not access the EHR system because the internal CA root certificate was missing from their trust stores.
- **Chain status:** Valid, but trust anchor missing on affected devices.
- **Key command:** `openssl verify -CAfile [rootCA.pem] [certfile]`
- **Lesson:** Trust store management is as important as certificate management. Infrastructure changes (like new subnets) require updating trust anchors.

---

## 4. Common Misconceptions

- **“If the chain is good, the connection should work.”** Not true—expiration or SAN mismatch will still cause failure.
- **“Revocation is always checked.”** Not if the cert is expired or the chain is broken; those checks come first.
- **“Wildcard certs cover all subdomains.”** They only match one level (e.g., *.example.com covers www.example.com, not a.b.example.com).
- **“Trust stores are static.”** They must be updated whenever new CAs or subnets are added.

---

## 5. Where This Shows Up

- **Websites:** Expired or misconfigured certs block users instantly.
- **Internal systems:** Trust store gaps or missing intermediates break service-to-service communication.
- **Cloud/DevOps:** Automated systems can fail if a cert or chain is missing or expired.
- **Healthcare/critical infrastructure:** Outages can impact patient care or business operations if PKI isn’t managed carefully.

---

## Mental Model

Think of PKI like a security checkpoint:
- **Expiration:** Is your ID still valid? If not, you’re turned away immediately.
- **Chain:** Can your ID be traced back to a trusted authority? If not, you’re denied entry.
- **SAN/Hostname:** Does your ID match the name on the guest list? If not, you’re not allowed in.
- **Trust store:** Is the authority that issued your ID recognized by the bouncer? If not, you’re not trusted, even if your ID is perfect.

Every step matters, and a failure at any point blocks access. Diagnosing PKI issues means checking each link in the chain, from the certificate itself to the trust anchor on the client.