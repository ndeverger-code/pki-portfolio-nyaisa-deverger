 # Lab 01: Explore and Duplicate a Certificate Template

**Student Name:**  Nyaisa Deverger
**Date Completed:** May 17, 2026
**Phase:** 2 | **Week:** 10  
**Submission Path:** `labs/week-10/lab-01-template-exploration.md`

---

## Pre-Lab Verification

Run the following on PKI-SRV01 before starting. Do not proceed until all checks pass.

```powershell
# Check 1 — CA service running
Get-Service -Name CertSvc
```

```
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services
```

```powershell
# Check 2 — CA responding
certutil -ping
```

```
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (31ms)
CertUtil: -ping command completed successfully.
```

```powershell
# Check 3 — Issuing CA cert in enterprise store
certutil -store -enterprise CA
```

```
CA "Intermediate Certification Authorities"
================ Certificate 0 ================
Serial Number: 5800000002f7714edc7f317c46000000000002
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
 NotBefore: 4/25/2026 7:26 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Certificate Template Name (Certificate Type): SubCA
Non-root Certificate
Template: SubCA
Cert Hash(sha1): 5137a597de2c3085ec5816c7f11edc18cfcdbaf8
No key provider information
  Provider = Microsoft Software Key Storage Provider
  Simple container name: CVI Issuing CA 1
  Unique container name: b52f658bb3f263e6f529f3a0187c63bc_f0a99c17-76d3-498a-97de-2992c06105fd
  ERROR: missing key association property: CERT_KEY_IDENTIFIER_PROP_ID
Signature test passed
CertUtil: -store command completed successfully.
```

**All checks passed:**
- [x] Yes
- [ ] No — describe the issue and how you resolved it:

---

## Part A — Explore Three Built-in Templates

Open the Certificate Templates console: **Run → certtmpl.msc**

### Template 1: User

| Field | Value |
|-------|-------|
| Template Display Name | User |
| Template Name (internal) | User |
| Minimum Supported CA | Windows 2000 |
| Validity Period | 1 year |
| Renewal Period | 6 weeks |

**General tab — notes:**

```
Publish certificate in Active Directory is enabled. This means issued certificates are
written back to the user object in AD, allowing other users and systems to discover and
use the certificate (e.g., for S/MIME email encryption lookups).
```

**Request Handling tab — Purpose:**

```
Signature and encryption
CSPs enabled: Microsoft Base Cryptographic Provider v1.0, Microsoft Enhanced Cryptographic Provider v1.0
Allow private key to be exported: Yes
```

**Subject Name tab — Subject name format:**

```
Built from information in Active Directory
Include e-mail name: Yes
Type of subject: User
```

**Extensions tab — Key Usage:**

```
Digital signature
Allow key exchange only with key encryption (key encipherment)
Critical extension: Yes
```

**Extensions tab — Application Policies (EKU):**

```
Encrypting File System
Secure Email
Client Authentication
```

---

### Template 2: Computer

| Field | Value |
|-------|-------|
| Template Display Name | Computer |
| Template Name (internal) | Machine |
| Validity Period | 1 year |
| Renewal Period | 6 weeks |

**Request Handling tab — Purpose:**

```
Signature and encryption
CSPs enabled: Microsoft Base Cryptographic Provider v1.0
Allow private key to be exported: No
```

**Subject Name tab:**

```
Built from information in Active Directory
Include e-mail name: No
Type of subject: Computer or other device
```

**Extensions tab — Key Usage:**

```
Digital signature
Allow key exchange only with key encryption (key encipherment)
Critical extension: Yes
```

**Extensions tab — Application Policies (EKU):**

```
Client Authentication
Server Authentication
```

---

### Template 3: Web Server

| Field | Value |
|-------|-------|
| Template Display Name | Web Server |
| Template Name (internal) | WebServer |
| Validity Period | 2 years |
| Renewal Period | 6 weeks |

**Request Handling tab — Purpose:**

```
Signature and encryption
CSPs enabled: Microsoft DH SChannel Cryptographic Provider, Microsoft RSA SChannel Cryptographic Provider
Allow private key to be exported: No
```

**Subject Name tab:**

```
Supplied in the request
Include e-mail name: No
Type of subject: Computer or other device
```

**Extensions tab — Key Usage:**

```
Digital signature
Allow key exchange only with key encryption (key encipherment)
Critical extension: Yes
```

**Extensions tab — Application Policies (EKU):**

```
Server Authentication
```

---

### Template Comparison

In your own words — what is the most significant difference between the User, Computer, and Web Server templates?

```
The biggest thing I noticed is that each template is scoped to a specific type of identity
and what that identity actually needs to do. The User template covers a person's account in
AD and includes EKUs for email, file encryption, and client authentication — basically
everything a user might need. The Computer template is similar in structure but is for
machine accounts, so it drops the email and EFS EKUs and adds Server Authentication, since
computers need to authenticate to services and vice versa. The Web Server template stood out
the most to me because it only has Server Authentication and it does not publish to AD at
all — it is meant for a service, not an account, and the longer 2-year validity reflects
that web server certificates are managed manually rather than auto-enrolled like user or
computer certificates.
```

Why does the Web Server template use "Supplied in the request" for the subject name rather than building it from Active Directory?

```
Web servers are identified by DNS hostnames — names like www.corp.cvilab.local or an
external FQDN — and those names are not attributes of any AD user or computer object.
AD can build a subject name from an account's sAMAccountName, UPN, or DNS host attribute,
but those values do not reliably match the hostname a web service needs in its certificate.
A single server might also host multiple services under different FQDNs, none of which
align to the machine account name in AD. By requiring the subject name to be supplied in
the request, the template puts responsibility on the administrator submitting the CSR to
provide the exact hostname the certificate needs to cover — which is the only party who
knows that value. This is also why web server certificates are not auto-enrolled: they
require deliberate human input at request time rather than automatic population from AD.
```

---

## Part B — Duplicate the Web Server Template

**Steps performed:**

1. Right-clicked the **Web Server** template → **Duplicate Template**
2. Selected compatibility settings:

   - Certification Authority: Windows Server 2003
   - Certificate Recipient: Windows XP / Server 2003

3. Opened the properties of the new duplicate template.

**General tab — changes made:**

| Setting | Original Value | New Value |
|---------|---------------|-----------|
| Template display name | Copy of Web Server | CVI-WebServer |
| Template name | Copy of Web Server | CVI-WebServer |
| Validity period | 2 years | 2 years |
| Renewal period | 6 weeks | 6 weeks |

**Rationale for validity period chosen:**

```
The 2-year validity period was kept from the original Web Server template. Web server
certificates are requested manually rather than auto-enrolled, so a longer validity reduces
the frequency of manual renewal operations. 2 years is a reasonable balance between
limiting exposure if the private key were ever compromised and avoiding excessive
operational overhead in a lab environment.
```

**Subject Name tab — changes made:**

| Setting | Change Made | Rationale |
|---------|-------------|-----------|
| Source of subject name | No change — kept as "Supply in the request" | Web server hostnames are not stored in AD; the requesting administrator must supply the correct DNS name in the CSR |
| Alternate subject name fields | No change — all unchecked | DNS SANs will be provided in the CSR at request time, not pre-populated by the template |

**Security tab — permissions confirmed:**

| Group / Account | Read | Write | Enroll | Autoenroll |
|-----------------|------|-------|--------|------------|
| Authenticated Users | Yes | Yes | Yes | No |
| Domain Admins | Yes | Yes | Yes | No |
| pki.admin | — | — | — | — |
| Enterprise Admins | — | — | — | — |

**Ensure Write and Enroll permissions are checked under Authenticated Users**

**Template saved:**
- [x] Yes — template visible in certtmpl.msc

---

## Part C — Inspect the Duplicate Template with certutil

Run the following command on PKI-SRV01:

```powershell
certutil -v -template CVI-WebServer
```

**Full output:**

```
  Name: Active Directory Enrollment Policy
  Id: {41635678-B3E8-4BD7-8FE7-D49A1E336991}
  Url: ldap:
34 Templates:

  Template[8]:
  TemplatePropCommonName = CVI-WebServer
  TemplatePropFriendlyName = CVI-WebServer
  TemplatePropEKUs =
1 ObjectIds:
    1.3.6.1.5.5.7.3.1 Server Authentication

  TemplatePropCryptoProviders =
    0: Microsoft RSA SChannel Cryptographic Provider
    1: Microsoft DH SChannel Cryptographic Provider

  TemplatePropMajorRevision = 64 (100)
  TemplatePropDescription = Computer
  TemplatePropSchemaVersion = 2
  TemplatePropMinorRevision = 3
  TemplatePropRASignatureCount = 0
  TemplatePropMinimumKeySize = 800 (2048)
  TemplatePropOID =
    1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.9126694.13796578

  TemplatePropV1ApplicationPolicy =
1 ObjectIds:
    1.3.6.1.5.5.7.3.1 Server Authentication

  TemplatePropEnrollmentFlags = 20 (32)
    CT_FLAG_AUTO_ENROLLMENT -- 20 (32)

  TemplatePropSubjectNameFlags = 4a000000 (1241513984)
    CT_FLAG_SUBJECT_ALT_REQUIRE_UPN -- 2000000 (33554432)
    CT_FLAG_SUBJECT_ALT_REQUIRE_DNS -- 8000000 (134217728)
    CT_FLAG_SUBJECT_REQUIRE_COMMON_NAME -- 40000000 (1073741824)

  TemplatePropPrivateKeyFlags = 1010000 (16842752)
    CTPRIVATEKEY_FLAG_ATTEST_NONE -- 0
    TEMPLATE_SERVER_VER_2003<<CTPRIVATEKEY_FLAG_SERVERVERSION_SHIFT -- 10000 (65536)
    TEMPLATE_CLIENT_VER_XP<<CTPRIVATEKEY_FLAG_CLIENTVERSION_SHIFT -- 1000000 (16777216)

  TemplatePropGeneralFlags = 20241 (131649)
    CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT -- 1
    CT_FLAG_MACHINE_TYPE -- 40 (64)
    CT_FLAG_ADD_TEMPLATE_NAME -- 200 (512)
    CT_FLAG_IS_MODIFIED -- 20000 (131072)

  TemplatePropSecurityDescriptor = O:S-1-5-21-3975454498-3980183307-2685672490-1105G:...

    Allow Enroll        CORP\Domain Admins
    Allow Enroll        CORP\Enterprise Admins
    Allow Enroll        NT AUTHORITY\Authenticated Users
    Allow Full Control  CORP\Domain Admins
    Allow Full Control  CORP\Enterprise Admins
    Allow Full Control  CORP\pki.admin
    Allow Write         NT AUTHORITY\Authenticated Users

  TemplatePropExtensions =
4 Extensions:

  Extension[0]:
    1.3.6.1.4.1.311.21.7: Flags = 0, Length = 31
    Certificate Template Information
        Template=1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.9126694.13796578
        Major Version Number=100
        Minor Version Number=3

  Extension[1]:
    2.5.29.37: Flags = 0, Length = c
    Enhanced Key Usage
        Server Authentication (1.3.6.1.5.5.7.3.1)

  Extension[2]:
    2.5.29.15: Flags = 1(Critical), Length = 4
    Key Usage
        Digital Signature, Key Encipherment (a0)

  Extension[3]:
    1.3.6.1.4.1.311.21.10: Flags = 0, Length = e
    Application Policies
        [1]Application Certificate Policy:
             Policy Identifier=Server Authentication

  TemplatePropValidityPeriod = 2 Years
  TemplatePropRenewalPeriod = 6 Weeks
CertUtil: -Template command completed successfully.
```

**From the certutil output — record the following:**

| Field | Value from certutil Output |
|-------|---------------------------|
| Template Name | CVI-WebServer |
| Template OID | 1.3.6.1.4.1.311.21.8.15886664.4298044.8996776.14853544.7902291.169.9126694.13796578 |
| Schema Version | 2 |
| Key Usage | Digital Signature, Key Encipherment (critical) |
| Enhanced Key Usage (EKU) | Server Authentication (1.3.6.1.5.5.7.3.1) |
| Validity Period | 2 Years |
| Subject Name flags | CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT — subject name is provided in the CSR, not built from AD |

---

## Reflection

**Why does AD CS require you to duplicate a built-in template rather than modifying it directly?**

Built-in templates are locked from direct modification because they are shipped by Microsoft and act as stable baselines that the CA infrastructure depends on. If you could edit them directly, a change on one CA could affect every CA in the forest that references the same template, and there would be no clean way to recover the original if something broke. Duplicating creates a new template object in AD that you fully own, so you can customize it however the environment requires without touching the original. It also makes it obvious from the template name and the CT_FLAG_IS_MODIFIED flag in certutil output that the template is a custom one, which helps with auditing and troubleshooting later.

**One setting in the template you found unexpected or would want to explore further:**

The CT_FLAG_AUTO_ENROLLMENT flag showing up in TemplatePropEnrollmentFlags surprised me. Since the template uses "Supply in the request" for the subject name, I would not expect autoenrollment to be possible — there is no way for a client to automatically know what hostname to put in the CSR. I would want to explore whether this flag is effectively overridden by CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT in the GeneralFlags, or whether it could cause unexpected enrollment attempts if the template were published to the CA without further review.

---

## Submission Checklist

- [x] Pre-lab verification completed and outputs recorded
- [x] Part A: All three templates explored with tab-level observations
- [x] Part A: Comparison question answered in own words
- [x] Part B: Duplicate template created as CVI-WebServer
- [x] Part B: All changes documented with rationale
- [x] Part C: certutil -template output pasted and key fields extracted
- [x] Reflection section completed
- [x] File saved as `lab-01-template-exploration.md`
- [ ] File committed to portfolio repo under `labs/week-10/`