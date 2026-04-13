# Lab 04 — Full Diagnostic Scenario: Incident Report

**Week 6 · PKI Incident Diagnosis & Troubleshooting**
**CVI PKI Career Pathway — Phase 1 Foundations**

---

## PKI Incident Report

**System:** Metro General EHR portal (ehr.metrogeneral.org)
**Reported:** Friday, 4:47 PM
**Author:** Nyaisa Deverger
**Status:** Diagnosis complete — pending remediation

---

### Executive Summary

Clinical staff on the new clinical subnet (10.22.0.0/24) could not access the EHR system due to TLS errors, while main office users were unaffected. The root cause was that the internal CA root certificate was not deployed to devices on the new subnet, so they did not trust the EHR server’s certificate. The fix is to distribute the root CA to all clinical subnet devices.

---

### Technical Findings

#### Finding 1 — Missing Root CA in Clinical Subnet Trust Stores

**Type:** Trust Store
**Severity:** Critical

**Detail:**
Devices on the clinical subnet do not have the Metro General Internal CA - G2 root certificate in their trust store, so they cannot validate the EHR server’s certificate.

**Evidence:**
- EHR works on main office network (root CA present), fails on clinical subnet (root CA missing).
- Clinical subnet was added after the last Group Policy push that distributed the root CA.

---

#### Finding 2 — Certificate and Chain Are Valid

**Type:** Certificate / Chain
**Severity:** None (for this incident)

**Detail:**
The EHR server’s certificate was renewed correctly, issued by the internal CA, and has a valid chain. There are no issues with the certificate fields or chain structure.

**Evidence:**
- Infrastructure team confirmed certificate was renewed and chain is valid.
- No scenario evidence of date, key, or issuer problems.

---

#### Finding 3 — Old Certificate Not Revoked

**Type:** Revocation
**Severity:** Low

**Detail:**
The previous certificate was replaced but not explicitly revoked. While this does not affect the current incident, it is a best practice to revoke replaced certificates to reduce risk.

**Evidence:**
- Scenario states the old certificate was replaced, but no mention of revocation.
- Revocation status is not relevant to the current outage, but is an operational gap.

---

### Diagnostic Steps

#### Step 1 — Retrieve

I would use:
```
openssl s_client -connect ehr.metrogeneral.org:443 -showcerts
```
If the server is only accessible from certain subnets, I would run this from a device on the clinical subnet. The expected output would show a valid certificate chain, but a trust error if the root CA is missing.

---

#### Step 2 — Parse

I would check:
- Subject and Issuer fields to confirm the certificate is from Metro General Internal CA - G2.
- Public Key field to confirm a new key was used (compare modulus or fingerprint).
- Validity dates to confirm the certificate is not expired or not yet valid.

Scenario confirms the certificate is new, issued by the correct CA, and has a 1-year validity window. No date or key issues are present.

---

#### Step 3 — Validate the Chain

Chain validation would succeed on main office devices but fail on clinical subnet devices. This confirms the problem is with the trust store on the clinical subnet, not the certificate or chain itself.

Command to test:
```
openssl verify -CAfile [path to root CA] [certfile]
```
On clinical subnet devices, the root CA is missing, so validation fails.

---

#### Step 4 — Check Revocation and Trust

I would check if the old certificate is still valid and whether it has been revoked, using the CA’s CRL or OCSP service. Revocation of the old certificate is not relevant to the current incident, but should be done for security best practices.

---

### Failures in Diagnostic Order

1. **Primary failure: Missing root CA in clinical subnet trust stores**
   - Type: Trust Store
   - Evidence: Clinical subnet added after last Group Policy push; only clinical subnet users are affected.

2. **Contributing factor: Old certificate not revoked**
   - Type: Revocation
   - Evidence: Scenario says old cert was replaced but not revoked; not directly related to current outage.

---

### Root Cause

The clinical subnet was added after the last Group Policy deployment of the internal CA root certificate. As a result, devices on the new subnet never received the root CA and cannot trust certificates issued by it. The process gap is a lack of automated trust store updates for new network segments.

---

### Remediation Steps

1. Immediate: Push the Metro General Internal CA - G2 root certificate to all devices on the clinical subnet using Group Policy or MDM.
2. Short-term: Verify all subnets and new devices receive trust store updates automatically.
3. Secondary: Revoke the old EHR server certificate to reduce risk of misuse.

---

### Prevention Recommendations

- Automate trust store updates for all new subnets and devices.
- Add a post-certificate-renewal checklist to confirm trust store coverage and revocation of replaced certificates.
- Implement monitoring to alert when trust store gaps or certificate validation errors occur.

---

### Lessons Learned

This incident revealed that trust store management must be included in onboarding new network segments. Relying on a one-time Group Policy push is not enough. The organization should automate trust store updates and add validation checks after any network or certificate changes.

---

## Reflection

The hardest part was tracing the failure to the trust store, not the certificate or chain. It was tempting to assume the new certificate was at fault, but following the framework made the real issue clear. In a real incident, I would double-check trust store deployment for all new devices and subnets.

---

*CVI PKI Career Pathway — Foundations Phase*