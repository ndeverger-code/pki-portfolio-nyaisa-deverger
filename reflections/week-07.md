# Week 7 Reflection

## Prompts

---

## Reflection

### 1. What did you learn this week?
I learned that TLS doesn't always terminate at the web server — it can end at a load balancer or
CDN in front of it. Analyzing servicenow.com showed me that the issuer is one of the best clues
for figuring out where TLS ends. I also learned how to use crt.sh to look at a company's full
certificate history.

### 2. What concept was most challenging?
The TLS termination analysis. The `Server: Apache` header made it look like Apache was handling
TLS, but the ACM issuer proved it had to be an AWS load balancer. I had to learn to trust the
issuer over the server header.

### 3. Where does this concept appear in real-world systems?
Almost everywhere. Banks, hospitals, cloud platforms — they all route traffic through a load
balancer or CDN before it reaches the actual server. It also shows up during cloud migrations
when organizations run multiple CAs at once, like ServiceNow was doing.

### 4. How would you explain this topic to a non-technical audience?
When you visit a secure website, your browser checks a digital badge to verify it's safe. That
check doesn't always happen at the website itself — it often happens at a security checkpoint in
front of it. This week I learned how to figure out where that checkpoint actually is.

### 5. What questions remain?
- After the AWS ALB terminates TLS, is traffic to the Apache origin re-encrypted or sent as plain HTTP?
- How do organizations like ServiceNow switch CAs without breaking anything during the transition?


---

## Professional Growth Check

- [x] I documented my work clearly and in my own words
- [x] I used structured formatting in my submission files
- [x] My commit message was meaningful and descriptive

---

*CVI PKI Career Pathway — Foundations Phase*

