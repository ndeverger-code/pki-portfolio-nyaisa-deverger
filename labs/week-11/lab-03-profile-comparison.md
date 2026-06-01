# Lab 03: Compare Certificate Profiles Side by Side

**Student Name:** Nyaisa Deverger
**Date Completed:** May 31, 2026
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-03-profile-comparison.md`

---

## Overview

Lab 03 is a synthesis exercise. You have issued three certificates across Weeks 10 and 11:

1. **TLS certificate** — issued from CVI-WebServer in Week 10, Lab 02
2. **Service account certificate** — issued from CVI-ServiceAccount in Week 11, Lab 01
3. **Code signing certificate** — issued from CVI-CodeSigning in Week 11, Lab 02

In this lab, you inspect all three certificates using certutil, document their settings in a structured comparison table, and write an analytical explanation of why the differences exist.

> **Prerequisite:** All three certificates must be present before you begin. If the Week 10 TLS certificate is no longer available, contact your instructor before proceeding.

---

## Pre-Lab — Locate All Three Certificates

If you can log into PKI-SRV01 as **CORP\pki.admin**, you are communicating with DC01 and the environment is ready.

### Step 1 — List All Certificates in the Personal Store

Open **PowerShell** on PKI-SRV01 as CORP\pki.admin and run:

```powershell
certutil -store My
```

This lists all certificates in the current user's personal store. You should see at least the code signing certificate from Lab 02 and the TLS certificate from Week 10. Scroll through the output and identify each certificate by its Template or Subject field.

**certutil -store My output:**

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

================ Certificate 1 ================
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
  Key Container = CVI Issuing CA 1
  Unique container name: b52f658bb3f263e6f529f3a0187c63bc_f0a99c17-76d3-498a-97de-2992c06105fd
  Provider = Microsoft Software Key Storage Provider
Signature test passed

================ Certificate 2 ================
Serial Number: 440000000395ff46642751bb82000000000003
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/17/2026 8:44 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=PKI-SRV01.corp.cvilab.local
Certificate Template Name (Certificate Type): Machine
Non-root Certificate
Template: Machine, Computer
Cert Hash(sha1): 4463a1a9a66692f76e49190a5fa5b900ede2a3bf
  Key Container = dde0604492fa91e4516bad250cf47793_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-Machine-5dfad181-2f91-46a7-9673-6ba4bd96e8ba
  Provider = Microsoft RSA SChannel Cryptographic Provider
Private key is NOT exportable
Encryption test passed
CertUtil: -store command completed successfully.
```

Certificate 0 (CVI-WebServer) matches the thumbprint from Week 10 Lab 02. The service account and code signing certificates are in user-store contexts, not the machine Personal store, so they do not appear here.

**Week 10 TLS certificate present:**
- [x] Yes — Thumbprint: 56195db06f786ce07a5eb9f52e83b92531d522fb
- [ ] No — contact instructor before proceeding

### Step 2 — Locate the Service Account Certificate

The service account certificate was enrolled via the `runas /user:CORP\svc.autoenroll mmc.exe` session in Lab 01. `svc.autoenroll` is an Active Directory user account, not a Windows service, so its certificate lives in that account's user Personal store — not in the machine Personal store shown above and not accessible via `certutil -store -service`.

The certificate was confirmed and its details recorded during that Lab 01 runas session. The thumbprint (`2d3f56c65ba9dae586111f826f9e5ab08a8cb6d8`) and all certificate fields used in Step 3 below come from that session.

### Step 3 — Record All Three Thumbprints

From the certutil outputs above, find and record the thumbprint for each certificate. The thumbprint is a 40-character hex string shown near the bottom of each certificate entry.

| Certificate | Template | Thumbprint |
|-------------|----------|------------|
| TLS (Week 10, Lab 02) | CVI-WebServer | 56195db06f786ce07a5eb9f52e83b92531d522fb |
| Service Account (Lab 01) | CVI-ServiceAccount | 2d3f56c65ba9dae586111f826f9e5ab08a8cb6d8 |
| Code Signing (Lab 02) | CVI-CodeSigning | 9bc05618794a936764a27d7e12fcc2120e3a72cc |

---

## Part A — Inspect All Three Certificates

### Step 1 — Inspect Each Certificate Using Its Thumbprint

Run the following command for each certificate, replacing the placeholder with the actual thumbprint. Remove any spaces from the thumbprint before running.

**Certificate 1 — TLS (CVI-WebServer)**

```powershell
certutil -store My "56195db06f786ce07a5eb9f52e83b92531d522fb"
```

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

Key Usage and EKU confirmed from MMC Details tab (Week 10 Lab 02): **Key Usage = Digital Signature, Key Encipherment (a0)** | **EKU = Server Authentication (1.3.6.1.5.5.7.3.1)**

---

***Certificate 2 — Service Account (CVI-ServiceAccount)**

The CVI-ServiceAccount certificate belongs to the `svc.autoenroll` user account. It must be inspected from within a session running as that account. The command below was run from a `runas /user:CORP\svc.autoenroll powershell.exe` session on PKI-SRV01:

```powershell
certutil -store -user My "2d3f56c65ba9dae586111f826f9e5ab08a8cb6d8"
```

**Full certutil output:**

```
My "Personal"
================ Certificate 0 ================
Serial Number: 44000000059271f2d220b1aef8000000000005
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/31/2026 5:37 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=Svc Autoenroll, OU=Service Accounts, DC=corp, DC=cvilab, DC=local
Non-root Certificate
Template: CVIServiceAccount
Cert Hash(sha1): 2d3f56c65ba9dae586111f826f9e5ab08a8cb6d8
CertUtil: -store command completed successfully.
```

Key Usage and EKU confirmed from certificate Details tab via runas MMC session: **Key Usage = Digital Signature** | **EKU = Client Authentication (1.3.6.1.5.5.7.3.2)**

---

**Certificate 3 — Code Signing (CVI-CodeSigning)**

```powershell
certutil -store -user My "9bc05618794a936764a27d7e12fcc2120e3a72cc"
```

**Full certutil output:**

```
My "Personal"
================ Certificate 0 ================
Serial Number: 44000000069be40163789196d9000000000006
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
 NotBefore: 5/31/2026 6:41 PM
 NotAfter: 4/25/2027 7:36 PM
Subject: CN=PKI Admin, OU=PKI Admins, DC=corp, DC=cvilab, DC=local
Non-root Certificate
Template: CVI Code Signing
Cert Hash(sha1): 9bc05618794a936764a27d7e12fcc2120e3a72cc
  Key Container = 7239eef9b351f59fb1631dd81d01d6ea_f0a99c17-76d3-498a-97de-2992c06105fd
  Simple container name: te-CVI Code Signing-57baff48-9201-4390-9908-f7fec156efe5
  Provider = Microsoft Enhanced Cryptographic Provider v1.0
Private key is NOT exportable
Signature test passed
CertUtil: -store command completed successfully.
```

Key Usage and EKU confirmed from Lab 02 (MMC Details tab + `Get-ChildItem -CodeSigningCert` output): **Key Usage = Digital Signature** | **EKU = Code Signing (1.3.6.1.5.5.7.3.3)**

---

### Step 2 — Complete the Comparison Table

Using the certutil outputs above, fill in every cell of the table below. Look for each field in the certutil output — Key Usage and EKU are shown in the Extensions section. No cells should be left blank.

> **Finding the EKU OID:** In the certutil output, look for the line starting with `1.3.6.1.5.5.7.3.` under Enhanced Key Usage — this is the OID. If you can't find it, run: `certutil -store My "<thumbprint>" | findstr OID`

| Field | TLS Certificate | Service Account Certificate | Code Signing Certificate |
|-------|----------------|----------------------------|--------------------------|
| Template Name | CVI-WebServer | CVI-ServiceAccount | CVI-CodeSigning |
| Subject | CN=PKI-SRV01.corp.cvilab.local | CN=Svc Autoenroll, OU=Service Accounts, DC=corp, DC=cvilab, DC=local | CN=PKI Admin, OU=PKI Admins, DC=corp, DC=cvilab, DC=local |
| Subject Source | Supplied in request | Build from AD | Build from AD |
| Issuer | CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local | CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local | CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local |
| Key Usage | Digital Signature, Key Encipherment | Digital Signature | Digital Signature |
| EKU | Server Authentication | Client Authentication | Code Signing |
| EKU OID(s) | 1.3.6.1.5.5.7.3.1 | 1.3.6.1.5.5.7.3.2 | 1.3.6.1.5.5.7.3.3 |
| Validity Period | 5/17/2026 – 4/25/2027 | 5/31/2026 – 4/25/2027 | 5/31/2026 – 4/25/2027 |
| Serial Number | 4400000004de07084d1f319d8e000000000004 | 44000000059271f2d220b1aef8000000000005 | 44000000069be40163789196d9000000000006 |
| Thumbprint | 56195db06f786ce07a5eb9f52e83b92531d522fb | 2d3f56c65ba9dae586111f826f9e5ab08a8cb6d8 | 9bc05618794a936764a27d7e12fcc2120e3a72cc |
| Request ID | 4 | 5 | 6 |

---

## Part B — Written Analysis

This section is the substance of Lab 03. Write in **prose paragraphs — not bullet points**. Each answer should explain the *why*, not just describe what you observed.

**1 — Key Usage**

Each of the three certificates has different Key Usage settings. Explain *why* each certificate has the Key Usage it has. For each, trace the setting back to the cryptographic operation the use case requires.

```
The TLS certificate has Digital Signature and Key Encipherment because a TLS server
certificate must support both of the dominant key exchange modes. In RSA key exchange
(used in TLS 1.2 and earlier), the client encrypts the pre-master secret using the
server's public key, and the server decrypts it to derive the session keys. That
operation is exactly what Key Encipherment authorizes. Digital Signature is required
because the server also signs portions of the TLS handshake to prove it holds the
private key, and it is the only key usage involved when cipher suites use Diffie-Hellman
key exchange instead. A web server certificate without both would break compatibility
with one class of cipher suites or the other.

The service account certificate has Digital Signature only because certificate-based
authentication is fundamentally a signing operation. When svc.autoenroll presents its
certificate, it uses its private key to sign a challenge issued by the verifying system.
The other side checks the signature with the public key and grants or denies access.
No encryption of keys or data occurs during this process. Key Encipherment would add
a capability the service account never exercises, and under the principle of least
privilege there is no justification for including it.

The code signing certificate also has Digital Signature only, which traces directly to
what Authenticode signing does at the cryptographic level. Signing a script means
hashing the file contents and using the private key to produce a signature over that
hash. Digital Signature is the precise key usage that authorizes this operation. Key
Encipherment would allow wrapping or unwrapping symmetric keys during a key exchange,
which code signing never does. Restricting the certificate to Digital Signature ensures
the private key cannot be used for anything other than producing signatures, which
limits the damage if the key is ever compromised.
```

Each certificate has a different EKU. Explain why each certificate has the EKU it has. For each, identify the relying party application that enforces that EKU and what it would do if the EKU were absent or wrong.

```
The TLS certificate has Server Authentication (1.3.6.1.5.5.7.3.1). The relying party
is the TLS client — a browser, application framework, or any process that opens a TLS
connection to the server. During the TLS handshake, the client validates the server's
certificate and checks for the Server Authentication EKU before completing the
connection. If the EKU is absent, the client rejects the certificate and the connection
fails with a certificate validation error. The EKU check is separate from chain
validation — a perfectly valid certificate that chains to a trusted root will still
be rejected if it lacks the Server Authentication OID.

The service account certificate has Client Authentication (1.3.6.1.5.5.7.3.2). The
relying party is whatever server or application svc.autoenroll is authenticating to —
typically a service endpoint using mutual TLS or certificate-based authentication. When
that service evaluates the presented certificate, it checks for Client Authentication
EKU. A certificate without it would be rejected as unauthorized for authentication
even if the chain is valid and the private key is correct. This also prevents a TLS
server certificate or code signing certificate from being misused for authentication;
each EKU acts as a guard at the application layer.

The code signing certificate has Code Signing (1.3.6.1.5.5.7.3.3). The relying party
is the Windows Authenticode verification subsystem — the component that checks
signatures on scripts and executables. When PowerShell evaluates a signed script under
an execution policy, it inspects the signing certificate for the Code Signing EKU.
Set-AuthenticodeSignature will refuse to sign with a certificate that lacks this EKU,
and Get-AuthenticodeSignature will report the signature invalid or untrusted if the
EKU is missing. This enforcement happens at the operating system level, meaning the
OS itself will not accept the certificate for this use case even if everything else
about the certificate is valid.
```

The TLS certificate uses "Supplied in request" for its subject. The service account and code signing certificates use "Build from Active Directory." Explain why the subject name source is different for the TLS certificate — what is it about the TLS use case that makes AD-supplied subject names impractical?

```
The TLS certificate uses "Supplied in request" because the subject of a TLS certificate
must reflect the server's DNS name or endpoint identity, and that information is not
stored in Active Directory in a usable form. TLS clients validate the server's identity
by matching the hostname they used to connect against the Subject Alternative Name
(SAN) in the certificate. A server might answer to multiple DNS names, or its external
DNS name might differ from its AD computer object name. If the CA built the subject
from AD, it would populate it with the computer object's CN — which may not match what
clients are actually connecting to. Supplying the subject in the request gives the
enrolling administrator explicit control over what names go into the certificate and
its SAN.

There is also a structural constraint: SANs for arbitrary DNS names cannot be derived
from AD computer attributes automatically. The person requesting the certificate knows
what hostnames the server needs to serve, and those values must be provided at
enrollment time. For the service account and code signing certificates, by contrast,
the identity that matters is already defined in AD — the account's CN, DN, or UPN —
so building from AD removes any chance of the subject being fabricated or mismatched,
and eliminates the need for manual input.
```

Imagine a single certificate that combined all three EKUs: Server Authentication, Client Authentication, and Code Signing. What security risk would this create? Be specific — what could an attacker do with a compromised private key from this combined certificate that they could not do with any one of the three individual certificates?

```
A certificate combining Server Authentication, Client Authentication, and Code Signing
would collapse three independent attack vectors into a single private key. An attacker
who compromises that key can impersonate PKI-SRV01 as a trusted HTTPS server to any
client that trusts the CA chain, authenticate as the certificate holder to any system
accepting mutual TLS or certificate-based client authentication, and sign arbitrary
scripts or executables that Windows Defender and execution policies will treat as
trusted publisher content. No single individual certificate enables all three at once.
The TLS certificate cannot sign code. The service account certificate cannot
impersonate a web server to a browser. The code signing certificate cannot authenticate
a client to a Kerberos-aware service. A combined certificate eliminates those
boundaries entirely.

The concrete damage from a single compromised key would be: rogue HTTPS servers that
browsers accept without a warning, silent lateral movement through services that trust
client certificates, and the ability to distribute signed malware that bypasses
execution controls across every system in the domain. The attacker does not need three
separate compromises — just one. This is exactly why separating certificate profiles
by purpose is a fundamental PKI design principle. Each certificate should carry only
the EKUs its use case requires, so that a compromised key has the smallest possible
blast radius.
```

**Which of the three certificates would you consider most critical to revoke quickly if the private key were compromised — and why?**

```
Out of the three, I would say the code signing certificate is the most urgent to
revoke if the private key got compromised. If someone stole the TLS certificate,
they could pretend to be PKI-SRV01, but they would need to be in the right position
on the network to actually pull that off, and it only affects people trying to connect
to that specific server. If someone stole the service account certificate, they could
log in as svc.autoenroll, but that account only has certain permissions, so the damage
is limited to what that account can do.

But if someone got the code signing certificate, they could sign any script or
executable they want, and Windows would treat it as trusted on every computer in the
domain. That is a much bigger problem because you cannot just lock down one server —
the signed files could already be out there running on machines everywhere. Revoking
the certificate would stop new things from being trusted, but anything already signed
and cached might still run. That is why speed matters so much here. The longer it
takes to revoke, the more damage can spread.
```

**What would you add to this comparison if you were presenting it to a security team evaluating the PKI environment?**

```
If I were showing this to a security team, I would not just hand them a table of
certificate fields. There are a few other things they would actually need to know
to make a real judgment about whether this PKI is secure.

First, they should know who is allowed to enroll for each certificate type and
whether anyone can do it without approval. A code signing certificate that anyone
with domain credentials can request without a manager signing off is a security
problem, even if the certificate itself looks fine. Second, all three certificates
showed up as non-exportable, which is good, but the team should know that means the
private key is protected from being copied out of the store — it does not mean someone
could not extract it from memory using the right tools. Third, revocation only matters
if the endpoints are actually checking it. If the CRL or OCSP is not set up right, or
if machines are not checking before they trust a certificate, then revoking something
does not actually stop it from being used. Fourth, they should have a way to get
alerted when a code signing certificate is issued, because that should not happen
very often and any unexpected enrollment is worth looking at. Finally, I would include
the request IDs — 4 for TLS, 5 for the service account, 6 for code signing — so they
could trace each certificate back to a specific CA log entry and confirm it was
legitimately requested.
```

---

## Submission Checklist

- [x] Pre-lab: All three thumbprints recorded in the table
- [x] Pre-lab: Service account cert located (personal store or svc.autoenroll store)
- [x] Part A: certutil output for all three certificates pasted in full
- [x] Part A: Comparison table fully completed — no blank cells
- [x] Part B: Key Usage analysis written in prose — traces each setting to a cryptographic operation
- [x] Part B: EKU analysis written in prose — identifies the relying party for each
- [x] Part B: Subject Name source difference explained with reasoning
- [x] Part B: Security question answered — specific risk identified for a combined-EKU certificate
- [x] Reflection completed
- [ ] File saved as `lab-03-profile-comparison.md`
- [ ] All three Week 11 lab files committed together with a single meaningful commit message
- [x] Request IDs for all three certificates recorded — needed for Week 12
- [ ] All three Week 11 labs committed with a single meaningful commit message
- [x] Request IDs for all three certificates recorded (TLS from Week 10, Service Account, Code Signing) — needed for Week 12