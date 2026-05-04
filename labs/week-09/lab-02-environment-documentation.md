# Lab 02: AD CS Console Exploration & CA Hierarchy Documentation
**Student Name:** Nyaisa Deverger

**Date Completed:** 05/04/2026

**Phase:** 2 | **Week:** 9  
**Submission Path:** `labs/week-09/lab-02-environment-documentation.md`

---

## Part A — AD CS Console Exploration (PKI-SRV01)

### CA Console Nodes — Observations

| Node | Contents / Observations |
|------|------------------------|
| Revoked Certificates | Empty — no certificates have been revoked yet |
| Issued Certificates | Empty — no certificates have been issued yet |
| Pending Requests | Empty — no pending certificate requests |
| Certificate Templates | 11 templates loaded after DC01 came online. Templates visible: Directory Email Replication, Domain Controller Authentication, Kerberos Authentication, EFS Recovery Agent, Basic EFS, Domain Controller, Web Server, Computer, User, Subordinate Certification Authority, Administrator |

### CA Properties — Key Settings

**General Tab**
- CA Name: CVI Issuing CA 1
- Computer Name: PKI-SRV01
- CA Certificate: Certificate #0
- Cryptographic Provider: Microsoft Software Key Storage Provider
- Hash Algorithm: SHA256

**Extensions Tab — CRL Distribution Points (CDP):**

```
Entry 1: C:\Windows\System32\CertSrv\CertEnroll\<CaName><CRLNameSuffix><DeltaCRLAllowed>.crl
  - Publish CRLs to this location: YES
  - Include in CDP extension of issued certificates: NO

Entry 2: ldap:///CN=<CATruncatedName><CRLNameSuffix>,CN=<ServerShortName>,...
  - Publish CRLs to this location: NO
  - Include in CRLs (Delta CRL locator): YES
  - Include in the CDP extension of issued certificates: YES
```

**Extensions Tab — Authority Information Access (AIA):**

```
Entry 1: C:\Windows\System32\CertSrv\CertEnroll\<ServerDNSName> <CaName><CertificateName>.crt
  - Include in the AIA extension of issued certificates: NO
  - Include in the OCSP extension: NO

Entry 2: ldap:///CN=<CATruncatedName>,CN=AIA,CN=Public Key Services,CN=Services,
         <ConfigurationContainer><CAObjectClass>
  - Include in the AIA extension of issued certificates: NO
  - Include in the OCSP extension: NO
```

**Storage Tab**
- Database Path: C:\Windows\System32\CertLog
- Log Path: C:\Windows\System32\CertLog

### Certificate Templates Console (certtmpl.msc)

Templates visible in the forest (list what you observed):

```
Source: Certificate Templates (DC01.corp.cvilab.local)

Administrator                          Authenticated Session
Basic EFS                              CA Exchange
CEP Encryption                         Code Signing
Computer                               Cross Certification Authority
Directory Email Replication            Domain Controller
Domain Controller Authentication       EFS Recovery Agent
Enrollment Agent                       Enrollment Agent (Computer)
Exchange Enrollment Agent (Offline request)
Exchange Signature Only                Exchange User
IPSec                                  IPSec (Offline request)
Kerberos Authentication                Key Recovery Agent
OCSP Response Signing                  RAS and IAS Server
Root Certification Authority           Router (Offline request)
Smartcard Logon                        Smartcard User
Subordinate Certification Authority    Trust List Signing
User                                   User Signature Only
Web Server                             Workstation Authentication

Total: 32 templates
```

---

## Part B — CA Hierarchy Verification (PKI-SRV01)

### Command: certutil -store -enterprise Root

```
Root "Trusted Root Certification Authorities"
================ Certificate 0 ================
Serial Number: 26373e51a6ab669340c47caef2232ce1
Issuer: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
 NotBefore: 4/25/2026 6:15 PM
 NotAfter: 4/25/2046 6:25 PM
Subject: CN=CVI Root CA, DC=corp, DC=cvilab, DC=local
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Cert Hash(sha1): b805e6ab548f6e7c57d3989f61de7fe6a51031d1
No key provider information
CertUtil: -store command completed successfully.
```

**What did you see?** (Subject, Issuer, Thumbprint — describe in your own words):

```
The enterprise Root store contains one certificate: CVI Root CA. The Subject and Issuer fields are
identical, which confirms this is a self-signed root CA — it vouches for itself and serves as the
trust anchor for the entire PKI hierarchy. The certificate is valid from April 25, 2026 through
April 25, 2046, giving it a 20-year validity period, which is standard for an offline root CA.
The SHA1 thumbprint is b805e6ab548f6e7c57d3989f61de7fe6a51031d1. There is no private key on this
machine, which is expected — the Root CA's private key stays on the offline Root-CA VM and should
never be present on PKI-SRV01.
```

### Command: certutil -store -enterprise CA

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
Template: SubCA, Subordinate Certification Authority
Cert Hash(sha1): 5137a597de2c3085ec5816c7f11edc18cfcdbaf8
No key provider information
  Provider = Microsoft Software Key Storage Provider
  Simple container name: CVI Issuing CA 1
Signature test passed
CertUtil: -store command completed successfully.
```

**What did you see?** (Subject, Issuer, Thumbprint — describe in your own words):

```
The enterprise Intermediate CA store contains one certificate: CVI Issuing CA 1. The Issuer is
CVI Root CA, confirming that the Root CA signed this certificate and authorized it to issue
certificates to end entities. The Subject is CVI Issuing CA 1 — this is the CA actively running
on PKI-SRV01. The certificate is valid for one year (April 25, 2026 through April 25, 2027),
 The SHA1 thumbprint is 5137a597de2c3085ec5816c7f11edc18cfcdbaf8.
The template used was SubCA (Subordinate Certification Authority), and the key is stored using
the Microsoft Software Key Storage Provider on this server.
```

---

## Part C — Active Directory Structure (DC01)

### Active Directory Users and Computers (dsa.msc)

**PKI Admins OU — accounts found:**

```
- Cert Manager   (User)
- PKI Admin      (User)
- PKI Admins     (Security Group — Global scope)
  Description: PKI administrators for the corp.cvilab.local CA hierarchy
  Members: Cert Manager, PKI Admin
```

**pki.admin account — group memberships:**

```
- Domain Admins   (corp.cvilab.local/Users)
- Domain Users    (corp.cvilab.local/Users) — Primary Group
- PKI Admins      (corp.cvilab.local/PKI Admins)
```

**cert.manager account — group memberships:**

```
- Domain Users    (corp.cvilab.local/Users) — Primary Group
- PKI Admins      (corp.cvilab.local/PKI Admins)
```

**Domain-joined computer accounts found:**

```
- PKI-SRV01   (Computer — Computers container)
```

### Active Directory Sites and Services (dssite.msc)

**Server registered under Default-First-Site-Name:**

```
Server: DC01
Domain: corp.cvilab.local
DC Type: GC (Global Catalog)
```

### Certificate Templates Console (certtmpl.msc on DC01)

Did templates appear here, confirming they are stored in AD?
- [ ] Yes
- [x] No — describe what happened:

> Windows returned *"cannot find 'certtmpl.msc'"* when run on DC01 because DC01 does not have
> the certificate management tools installed — those tools only live on PKI-SRV01. The templates
> are still stored in AD though: when `certtmpl.msc` was opened on PKI-SRV01, the source line
> read **"Certificate Templates (DC01.corp.cvilab.local)"**, confirming it was pulling the list
> from DC01's Active Directory.

---

## Part D — Environment Summary Write-Up

> Write this section in your own words. Cover all five areas. Aim for clarity over length.

### 1. Environment Topology

Three VMs make up this lab. DC01 (IP 192.168.10.10) is the domain controller — it handles logins and Active Directory for `corp.cvilab.local`. PKI-SRV01 is where the Certificate Authority runs. Root-CA is the offline root CA that only gets powered on when needed. DC01 must start first since the other two depend on it.

### 2. CA Hierarchy

There are two CAs. CVI Root CA is the top of the chain — it signed its own certificate and is kept offline so its key can never be stolen. CVI Issuing CA 1 runs on PKI-SRV01 and is the one that actually hands out certificates. CVI Root CA signed CVI Issuing CA 1's certificate, which is what gives it the authority to issue.

### 3. Certificate Templates

CVI Issuing CA 1 has 11 templates published to it. Templates are basically blueprints — they define what type of certificate gets issued and who can ask for one. For example, the **Web Server** template issues certificates for HTTPS websites, and the **Computer** template issues certificates so domain machines can prove their identity on the network.

### 4. Active Directory Structure

There is a PKI Admins OU in Active Directory with two accounts and a security group. `pki.admin` is in Domain Admins, so it has full admin rights across the domain plus PKI access. `cert.manager` is only in the PKI Admins group, meaning it can manage certificates but nothing else on the domain. This keeps permissions tight — each account only has what it needs.

### 5. One Thing I Found Interesting or Unexpected

I found it interesting that `certtmpl.msc` would not open on DC01, even though the certificate templates are actually stored in DC01's Active Directory. The tool failed because it requires the AD CS management tools to be installed, and DC01 does not have them — only PKI-SRV01 does. So the templates live on DC01, but you can only view them from PKI-SRV01. That separation was not something I expected going in.

---

## Submission Checklist

- [x] Part A: All five console nodes documented
- [x] Part A: CA Properties (CDP, AIA, Storage) recorded
- [x] Part A: Certificate Templates console observed
- [x] Part B: certutil -store -enterprise Root output included
- [x] Part B: certutil -store -enterprise CA output included
- [x] Part C: PKI Admins OU and both accounts documented
- [x] Part C: Sites and Services server noted
- [x] Part C: certtmpl.msc confirmed templates in AD
- [x] Part D: All five summary areas completed in own words
- [ ] File committed to portfolio repo at `labs/week-09/lab-02-environment-documentation.md`