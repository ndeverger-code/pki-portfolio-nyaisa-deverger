# Week 01 Mini Lab — Trust Chain Validation

## Screenshot Evidence

Capture a screenshot of the Certification Path (certificate chain) from your browser.

Save it as:

assets/screenshots/week-01/trust-chain-validation.png

Embed the screenshot below:

![Trust Chain Validation](../../assets/screenshots/week-01/trust-chain-validation.png)


## Website Information

**Website inspected:**  
https://www.reddit.com/

---

## Certificate Chain Breakdown

**Leaf (Server) Certificate**  

Common Name (CN): *.reddit.com

**Intermediate Certificate Authority**
DigiCert Global G2 TLS RSA SHA256 2020 CA1

**Root Certificate Authority (Trust Anchor)**
DigiCert Global Root G2

---

## Trust Anchor Verification

Is the Root CA marked as trusted by your system?

Yes

If yes, explain where that trust comes from (OS/browser root store).
 DigiCert Global Root G2 is located in the root store

If no, explain what warning or behavior occurred.

---

## Observations

Document three observations about the certificate.

### Observation 1
Trust flows from the root certificate (trusted by the system) down through the intermediate CA to the server certificate which 
allows the browser to authenticate

### Observation 2
The Root and Intermediate are both issued by the same authority - DigiCert

### Observation 3
I observed that because the root CA is included in the browser’s trusted root certificate store, the browser accepts the entire chain as valid.

---

## Reflection

In 3–5 sentences, explain:
- Why the Root certificate is called a trust anchor
  The root certificate is called a **trust anchor** because the operating system or browser already trusts it by default. It is stored in the system’s trusted certificate store and acts as the starting point for verifying other certificates
- How validation walks the certificate chain
  1. The browser checks that the server certificate was signed by the intermediate CA DigiCert Global G2 TLS RSA SHA256 2020 CA1
  2. It then verifies that the intermediate certificate was signed by the root CA DigiCert Global Root G2.\
  3. The browser accepts the entire certificate chain
  4. The browser confirms the website is valid and secure

2. - What would happen if the Root CA were not trusted
  I would be advised that the connection is not private via the website browser

Use your own words.
