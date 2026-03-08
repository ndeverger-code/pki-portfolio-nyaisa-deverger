# Week 01 Lab — Key Pair Generation

## Screenshot Evidence

If using OpenSSL:
1. Capture a screenshot showing:
  - The command used to generate the private key
  - The command used to extract the public key
2. Save it as:

**assets/screenshots/week-01/keypair-generation.png**

3. Embed the screenshot below:

**![Key Pair Generation](../../assets/screenshots/week-01/keypair-generation.png)**

If using a browser-based generator, capture the generated key pair screen (redact sensitive portions of the private key before committing).

---

## Key Identification
**Which file is the public key?**  
- `public.key` (extracted from the private key using `openssl rsa -in private.key -pubout -out public.key`)

**Which file is the private key?**  
- `private.key` (generated using `openssl genrsa -out private.key 2048`)

---

## Key Properties
- **What makes the public key safe to share:**  
  The public key can be shared because it cannot be used to impersonate you or decrypt messages without the private key

- **What makes the private key sensitive:**  
  The private key must remain secret because anyone with it can impersonate you, decrypt messages meant only for you, and break trust

---

## Security Scenario
**What would happen if someone obtained your private key?**  

If someone got your private key:  
- They could pretend to be you (**identity**)
- They could send messages or log in as you (**impersonation**)
- Others could no longer trust your communications (**trust**)

---

## Observations
### Observation 1
- Generating the private key was quick, and the public key can be easily derived

### Observation 2
- The key size (2048 bits) makes it secure

### Observation 3
- The private key is longer and sensitive, while the public key can be shared safely and is used only for verification or encryption

---
## Reflection
**In 3–5 sentences, explain: Why must the private key remain secret in a PKI system?**  
The private key must remain secret in a PKI system because it proves your identity. If someone else has it, they could pretend to be you and break trust. The public key can be shared safely because it cannot be used to impersonate you. In PKI, your identity is tied to possession of the private key

Focus on how identity is tied to possession of the private key.
