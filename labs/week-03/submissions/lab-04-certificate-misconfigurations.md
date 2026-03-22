
# Lab 04 — Detect Certificate Misconfigurations

## Overview
Briefly describe what this lab was about in your own words.
What PKI concept were you investigating?

---

## Scenario 1 — Missing Subject Alternative Name

**Would modern browsers trust this certificate?**
[Your answer]

**Analysis:**
[Explain why SAN is required, why CN is not sufficient, and what error users would see]

---

## Scenario 2 — Incorrect Extended Key Usage

**Would a browser accept this certificate for a web server?**
[Your answer]

**Analysis:**
[Explain what EKU defines, what value is required for HTTPS, and what error users would see]

---

## Scenario 3 — Expired Certificate

**What happens if this certificate is used today?**
[Your answer]

**Analysis:**
[Explain why expiration fails validation, why lifecycle management matters, and what users would see]

---

## Scenario 4 — Missing Intermediate Certificate

**Can the browser build a complete trust chain?**
[Your answer]

**Analysis:**
[Explain why the full chain must be served, what happens when the intermediate is missing, and how this is fixed]

---

## Key Takeaway
What is the most important thing you learned about certificate misconfigurations from this lab?
