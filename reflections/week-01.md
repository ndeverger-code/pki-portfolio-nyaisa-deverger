# Week X Reflection

Submit this in your portfolio repository:
reflections/week-XX.md

---

## Prompts

## 1. What did you learn this week?
- What PKI is and what it is not

## 2. What concept was most challenging?
- Understanding how trust moves from the root certificate down to the server certificate

## 3. Where does this concept appear in real-world systems?
- It appears in all secure websites and apps that use HTTPS, like online banking, social media, and e-commerce sites.

## 4. How would you explain this topic to a non-technical audience?
- It’s like a chain of ID checks: the website shows an ID, a middle authority verifies it, and a trusted authority everyone knows confirms it’s valid. If the chain checks out, the website is safe.

## 5. What questions remain?

---

## Professional Growth Check

- Did you document clearly? - Yes
- Did you use structured formatting? Yes
- Was your commit message meaningful? Yes

 # Week 1 Reflection Prompts



---

## 1. In your own words, what is digital trust?

**Digital trust** is knowing that a website, service, or person online is really who they say they are.

---

## 2. What is the difference between identity and authentication?

- **Identity** is who you say you are, like your username or account.  
- **Authentication** is proving it, such as entering a password or using a security code.  

Identity says “this is me,” while authentication shows “this really is me.” Both are needed for digital trust.

---

## 3. Explain public key vs private key like you’re teaching a beginner

In **PKI**, there are two keys that work together: a **private key** and a **public key**.  
- The **private key** is secret and belongs only to you. It proves your identity and can be used to decrypt messages or sign documents.  
- The **public key** can be shared with anyone. Other people use it to send you encrypted messages or verify your signatures.  

The private key must stay secret; the public key is meant to be shared.

---

## 4. What does a certificate *prove*? What does it *not* prove?

- A **certificate proves** that a public key belongs to a specific person and/or website, and that a trusted certificate authority (CA) has verified it. It shows that the server presenting the certificate is legitimate and can be trusted.  
- A certificate **does not prove** that the website is safe, reliable, or trustworthy in terms of content. A site could have a valid certificate but still be used for scams.

---

## 5. Where did you see PKI in your daily life this week?

I saw PKI in many places this week, mostly while using websites with HTTPS. For example:  
- At work when updating SANS information for certificate renewals  
- Amazon  
- School when logging onto the school site  

---

## 6. What concept is still unclear?

The concept that is still unclear to me is **how the browser actually walks the certificate chain step by step** when multiple intermediate CAs exist between the server certificate and the root certificate. I understand the idea of a chain of trust, but sometimes multiple intermediate certificates are involved, and I am unsure how the system chooses which path to trust or what happens if one intermediate certificate expires or is revoked.
