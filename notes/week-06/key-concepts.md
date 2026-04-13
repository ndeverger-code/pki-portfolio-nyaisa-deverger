# Week 6 Lesson Notes — PKI Incident Diagnosis & Troubleshooting

## 1. Core Concept

This week was about figuring out why a TLS connection fails and how to track down the problem step by step. The main idea is that certificates can break for different reasons — they can expire, the chain can be missing pieces, or the name on the cert might not match the site. To diagnose it, you follow a framework: grab the certificate, read its details, check the chain, and look at revocation. In the lab I did, the certificate on expired.badssl.com had expired back in 2015, so the connection failed right away.

---

## 2. Why It Matters

If a certificate expires on a website or a company's internal system, nobody can connect to it securely. It just stops working. This happens a lot in the real world — companies forget to renew their certificates and their sites go down. It's one of the most common and preventable security outages out there.

---

## 3. Technical Breakdown

- **Validity period:** Every certificate has a start date (Not Before) and an end date (Not After). If today's date is past the Not After date, the certificate is expired and gets rejected.
- **What gets checked first:** The expiration date is checked before anything else. If the cert is expired, the browser doesn't even bother checking if it was revoked — it's already done.
- **Chain validation:** Even if one thing is clearly wrong (like expiration), you should still check the whole chain to make sure there aren't other problems hiding underneath. In my lab, the chain was fine — leaf → intermediate → root — so expiration was the only issue.
- **Commands I used:**
  - `openssl s_client -connect <host>:443 -showcerts` — connects to the server and pulls down the certificate
  - `openssl x509 -in cert.pem -text -noout` — reads all the details of the certificate
  - `openssl x509 -in cert.pem -noout -dates` — quick way to just see the start and end dates
  - `grep -A1 "OCSP"` — finds the OCSP URL to check if revocation info is available

---

## 4. Common Misconceptions

- **"You need to check revocation for an expired certificate"** — Nope. Expiration gets checked first. If the cert is expired, the connection is already rejected before revocation is even looked at.
- **"If the chain is good, the connection should work"** — Not true. The chain being complete doesn't help if the leaf certificate itself is expired or has the wrong name on it.
- **"Shorter certificate lifespans are always better"** — They limit damage if a key gets stolen, but if you don't have automatic renewal set up, you're more likely to have the cert expire on you.

---

## 5. Where This Shows Up

- **Websites:** If a site's cert expires, your browser throws up a big warning and blocks the page.
- **Company systems:** Internal tools and APIs use certificates too. When those expire, services stop talking to each other and things break behind the scenes.
- **Cloud services:** Some cloud platforms auto-renew certs for you, but if someone set one up manually, it can still expire without warning.
- **DevOps pipelines:** Build and deploy systems that pull code from HTTPS sources will fail if those sources have expired certificates.

---

## Mental Model

Think of a certificate's expiration date like an ID card. Before anyone checks your name or photo, they flip it over and check if it's expired. If it is, nothing else matters — you're not getting in. That's how TLS works too. Expiration is the very first trust check: **is this identity still current?** Everything else — who issued it, what names it covers, whether it was revoked — comes after.