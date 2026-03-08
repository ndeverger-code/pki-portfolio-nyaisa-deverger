# Week 01 — Key Concepts

## PKI Explanation (3–5 Sentences)

**Public Key Infrastructure (PKI)** is a system that helps computers and people communicate safely over the internet. It solves the problem of **how to know if a website or user is really who they say they are** and how to **keep information private**.

### Focus on:

- **The problem PKI solves**  
  PKI solves the problem of knowing if a website or person online is really who they say they are and making sure that messages sent over the internet can’t be read or changed by someone else.

- **The role of keys**  
  PKI uses **two keys**:  
  - A **private key**, which is secret and proves your identity.  
  - A **public key**, which can be shared with anyone and is used to send encrypted messages or check signatures.

- **The role of certificates**  
  A **certificate** is like an ID card for a website or user. It contains the public key and is signed by a trusted certificate authority (CA) to show it’s real.

- **How trust is established**  
  Trust works because the browser or system already **trusts the root certificate** from the CA. When a website shows a certificate, the browser checks the chain of trust from the root CA down to the site. If everything is correct, the browser knows the site is safe and can send information securely.





  
