# Lab 02: Issue Your First Certificate from a Custom Template

**Student Name:** Nyaisa Deverger
**Date Completed:** May 17, 2026
**Phase:** 2 | **Week:** 10  
**Submission Path:** `labs/week-10/lab-02-first-issuance.md`

---

## Pre-Lab Verification

Run on PKI-SRV01 before starting.

```powershell
Get-Service -Name CertSvc
certutil -ping
```

**CertSvc status:** Running  
**CA responding (certutil -ping):**
- [x] Yes
- [ ] No — action taken:

**CVI-WebServer template visible in certtmpl.msc (from Lab 01):**
- [x] Yes
- [ ] No — complete Lab 01 before proceeding

---

## Part A — Publish the Template to the CA

The CVI-WebServer template exists in Active Directory but is not yet published to CVI Issuing CA 1. Publishing it makes it available for certificate requests.

**Steps performed on PKI-SRV01:**

1. Opened **certsrv.msc**
2. Expanded **CVI Issuing CA 1** → right-clicked **Certificate Templates** → **New → Certificate Template to Issue**
3. Selected **CVI-WebServer** from the list
4. Clicked **OK**

**CVI-WebServer template now visible under Certificate Templates node:**
- [x] Yes
- [ ] No — describe what happened:

```
N/A
```

**Screenshot or description of the Certificate Templates node showing CVI-WebServer:**

```
certsrv.msc → CVI Issuing CA 1 → Certificate Templates node shows CVI-WebServer listed
at the top of the template list with Intended Purpose: Server Authentication.
Other templates visible include Directory Email Replication, Domain Controller
Authentication, Kerberos Authentication, EFS Recovery Agent, Basic EFS, Domain
Controller, Web Server, Computer, User, Subordinate Certification Authority,
and Administrator.
```

---

## Part B — Request the Certificate via MMC

**Steps performed on PKI-SRV01, logged in as CORP\pki.admin:**

1. Opened **mmc.exe** → **File → Add/Remove Snap-in**
2. Added **Certificates** snap-in
3. Selected: **Computer account** (Local computer)
   - *Note: Initially selected "My user account" — CVI-WebServer template did not appear. Had to re-open MMC and select Computer account instead. The template's security permissions grant enrollment to computer accounts, not user accounts.*
4. Navigated to **Personal → Certificates**
5. Right-clicked → **All Tasks → Request New Certificate**
6. Proceeded through the Certificate Enrollment wizard

**Certificate Enrollment wizard — enrollment policy selected:**

```
Active Directory Enrollment Policy
```

**Templates shown in the wizard:**

```
- Computer          — STATUS: Available   (auto-selected)
- CVI-WebServer     — STATUS: Available   (auto-selected)
- Administrator     — STATUS: Unavailable (insufficient permissions)
- Authenticated Session — STATUS: Unavailable (insufficient permissions)
(additional templates hidden; "Show all templates" checkbox was checked)
```

**CVI-WebServer template visible:**
- [x] Yes
- [ ] No — troubleshooting steps taken:

**Subject name entered (if prompted):**

```
Subject name was auto-populated from Active Directory — no manual entry required.
The CA built the subject from the computer object: PKI-SRV01.corp.cvilab.local
```

**Certificate request submitted:**
- [x] Yes — certificate issued immediately
- [ ] Yes — certificate pending manager approval
- [ ] No — error encountered:

```
N/A — certificate issued without delay. PKI-SRV01.corp.cvilab.local now appears
in Personal → Certificates, issued by CVI Issuing CA 1, Certificate Template: CVI-WebServer,
Intended Purpose: Server Authentication. Expiration: 4/25/2027.
```

---

## Part C — Inspect the Issued Certificate

### In the MMC Certificates Snap-in

Navigate to the Personal → Certificates store and double-click the issued certificate.

**General tab:**

| Field | Value |
|-------|-------|
| Issued to | PKI-SRV01.corp.cvilab.local |
| Issued by | CVI Issuing CA 1 |
| Valid from | 5/17/2026 |
| Valid to | 4/25/2027 |

**Details tab — record the following fields:**

| Field | Value |
|-------|-------|
| Serial Number | 4400000004de07084d1f319d8e000000000004 |
| Signature Algorithm | sha256RSA |
| Subject | PKI-SRV01.corp.cvilab.local |
| Key Usage | Digital Signature, Key Encipherment (a0) |
| Enhanced Key Usage | Server Authentication (1.3.6.1.5.5.7.3.1) |
| Subject Alternative Name (if present) | Other Name: Principal Name=PKI-SRV01$@corp.cvilab.local; DNS Name=PKI-SRV01.corp.cvilab.local |
| Thumbprint | 56195db06f786ce07a5eb9f52e83b92531d522fb |

---

### Via certutil

Export the certificate thumbprint from the Details tab, then run:

```powershell
certutil -store My "<thumbprint>"
```

Replace `<thumbprint>` with the thumbprint value (no spaces).

**Full certutil output:**

```
My "Personal"
================ Certificate 0 ================
Serial Number: 4400000004de07084d1f319d8e000000000004
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/17/2026 8:44 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=PKI-SRV01.corp.cvilab.local
Non-root Certificate
Template: CVI-WebServer
Cert Hash(sha1): 56195db06f786ce07a5eb9f52e83b92531d522fb
  Key Container = c331eac02d8b30f9c3b8d755edc095ac_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVI-WebServer-d1955e87-3348-4fc3-b3ce-5e86db91ec46
  Provider = Microsoft RSA SChannel Cryptographic Provider
Private key is NOT exportable
Encryption test passed
CertUtil: -store command completed successfully.
```

---

### In certsrv.msc — Issued Certificates Node

Navigate to **certsrv.msc → CVI Issuing CA 1 → Issued Certificates**.

**Does the certificate appear in the Issued Certificates node?**
- [x] Yes

**Record from the Issued Certificates node:**

| Column | Value |
|--------|-------|
| Request ID | 4 |
| Requester Name | CORP\PKI-SRV01$ |
| Certificate Template | CVI-WebServer |
| Issued Common Name | PKI-SRV01.corp.cvilab.local |
| Certificate Expiration Date | 4/25/2027 7:36 PM |

---

## Part D — Write-Up: The Issuance Workflow

Describe the full certificate issuance workflow in your own words. Cover:

1. What happened in Active Directory when you published the template
2. What the MMC Certificate Enrollment wizard sent to the CA
3. What the CA evaluated before issuing the certificate
4. Where the issued certificate was placed and why

```
Before a certificate can be issued, the template has to be published to the CA — it's
not enough for it to exist in Active Directory. Once I published CVI-WebServer through
certsrv.msc, the CA could see it and it was an option for enrollment. When I opened
MMC and launched the Certificate Enrollment wizard, it contacted the CA through the Active
Directory Enrollment Policy endpoint and pulled back a list of templates my computer
account was permitted to request. CVI-WebServer showed up as available because the
template permissions allow computer accounts to enroll. The wizard built a certificate
request using the template's settings — key size, key usage, validity period — and
submitted it to CVI Issuing CA 1.

The CA evaluated the request by checking that the requester (PKI-SRV01$) had the
Enroll permission on the template, that the request matched the template's requirements,
and that no manager approval was required. Since all of that checked out, the CA signed
and issued the certificate immediately. The certificate was placed back into the
computer's Personal certificate store automatically, which is where Windows expects to
find machine certificates used for things like TLS. I could then see the same certificate
from two directions — in the MMC store on the client side and in the Issued Certificates
node on the CA side — which confirmed the full round trip worked.
```

**One thing about the issuance process that you did not expect or want to understand better:**

```
I didn't expect the snap-in account type to matter so much. When I selected "My user
account" in MMC, the CVI-WebServer template didn't show up at all — it only appeared
after I switched to "Computer account." That made me realize template permissions are
scoped specifically to the principal type (user vs. computer), not just to a group or
individual. I want to understand better how to configure a template so it can be
requested by both user and computer accounts if needed.
```

---

## Submission Checklist

- [x] Pre-lab verification completed
- [x] Part A: CVI-WebServer template published to CVI Issuing CA 1
- [x] Part A: Template visible in certsrv.msc Certificate Templates node — confirmed
- [x] Part B: Certificate requested via MMC — request submitted
- [x] Part B: Enrollment wizard observations documented
- [x] Part C: Certificate details recorded from MMC (General + Details tabs)
- [x] Part C: certutil -store My output pasted
- [x] Part C: Certificate confirmed in certsrv.msc Issued Certificates node
- [x] Part D: Issuance workflow write-up completed in own words
- [ ] File saved as `lab-02-first-issuance.md`
- [ ] File committed to portfolio repo under `labs/week-10/`