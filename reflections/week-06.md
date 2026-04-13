 # Week 6 Reflection

## Prompts

---

## Reflection

### 1. What did you learn this week?
This week, I learned how to systematically diagnose TLS failures using real-world PKI troubleshooting scenarios. I practiced identifying and resolving issues like expired certificates, broken chains (missing intermediates), hostname/SAN mismatches, and trust store misconfigurations. The labs reinforced a step-by-step diagnostic framework: retrieve the certificate, parse its details, validate the chain, and check revocation/trust. I also learned how small misconfigurations can cause major outages and the importance of trust store management.

### 2. What concept was most challenging?
The most challenging concept was diagnosing trust store issues, especially when the certificate and chain appeared valid. It was easy to assume the certificate was at fault, but the real problem was missing trust anchors on certain devices. Tracing failures to the trust store (rather than the certificate itself) required careful analysis and following the diagnostic steps without jumping to conclusions.

### 3. Where does this concept appear in real-world systems?
These concepts are critical in any environment that uses TLS/SSL for secure communications—public websites, internal portals, APIs, and enterprise applications. Outages caused by expired certificates, missing intermediates, or trust store gaps are common in hospitals, banks, and cloud services. The labs simulated real incidents like EHR outages, staff portal access failures, and misconfigured web servers.

### 4. How would you explain this topic to a non-technical audience?
I would say: "Certificates are like digital ID cards for websites and servers. If the ID is expired, incomplete, or doesn’t match the name on the door, your computer won’t trust it and blocks access. Sometimes, the problem isn’t with the ID itself, but with your computer not having the right list of trusted authorities. Keeping these lists updated and making sure certificates are valid is essential for secure access."

### 5. What questions remain?
- What are the best practices for automating trust store updates across large, dynamic networks?
- How do organizations monitor for and respond to certificate validation errors in real time?
- What tools exist to simplify diagnosing complex PKI failures in production environments?


## Professional Growth Check

- [Y] I documented my work clearly and in my own words
- [Y] I used structured formatting in my submission files
- [Y] My commit message was meaningful and descriptive


---