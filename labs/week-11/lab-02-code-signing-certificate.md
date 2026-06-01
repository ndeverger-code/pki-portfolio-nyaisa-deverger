# Lab 02: Issue a Code Signing Certificate

**Student Name:**  Nyaisa Deverger
**Date Completed:**  5/31/2026
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-02-code-signing-certificate.md`

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as **CORP\pki.admin**, you are communicating with DC01 and the environment is ready. Proceed to Part A.

---

## Part A — Design the CVI-CodeSigning Template

### Step 1 — Open the Certificate Templates Console

1. Press **Win + R**, type `certtmpl.msc`, and press **Enter**
2. The Certificate Templates console opens showing all templates installed on this CA

### Step 2 — Duplicate the Code Signing Template

1. Scroll through the list to find the built-in **Code Signing** template
2. Right-click **Code Signing** → select **Duplicate Template**
3. A new template Properties window opens — this is your working copy

> **Why Code Signing and not User?** The built-in Code Signing template already has the correct Key Usage (Digital Signature only) and EKU (Code Signing only) pre-configured. Starting from User would require removing multiple EKUs and introduces the risk of leaving incorrect settings in place.

**Source template duplicated:** Code Signing

### Step 3 — Set Compatibility Settings

1. Click the **Compatibility** tab
2. Set **Certification Authority** to: `Windows Server 2012 R2`
3. Set **Certificate Recipient** to: `Windows 8.1 / Windows Server 2012 R2`
4. Click **OK** on any informational dialog that appears

**Compatibility settings:**
- Certification Authority: Windows Server 2012 R2
- Certificate Recipient: Windows 8.1 / Windows Server 2012 R2

### Step 4 — Set the Template Name (General Tab)

1. Click the **General** tab
2. Change **Template display name** to: `CVI Code Signing`
3. Confirm the **Template name** (internal) auto-fills as `CVI-CodeSigning`

**Template names:**

| Field | Value |
|-------|-------|
| Template display name | CVI Code Signing |
| Template name (internal) | CVI Code Signing |

### Step 5 — Configure Key Usage

1. Click the **Extensions** tab
2. Select **Key Usage** in the list → click **Edit**
3. Confirm only **Digital Signature** is checked. Uncheck anything else if present
4. Check **Make this extension critical**
5. Click **OK**

| Key Usage | Included? | Reason |
|-----------|-----------|--------|
| Digital Signature | Yes | Code signing requires computing a signature over the code hash. Digital Signature is the only Key Usage that authorizes this operation. |
| Key Encipherment | No | Key Encipherment is for encrypting symmetric keys during TLS handshakes. Code signing never wraps or unwraps encryption keys, so this flag does not belong here. |
| Non-Repudiation | No | Windows Authenticode does not require Non-Repudiation. Including it would grant more key usage than a code signing certificate actually needs. |

**Explanation of Key Usage decision:**

```
The whole point of a code signing certificate is to sign code, and Digital Signature
is the key usage that allows exactly that. The private key signs a hash of the file
and anyone who wants to verify it uses the public key to check the signature. That
is all this certificate needs to do. Key Encipherment is for encrypting keys in
things like TLS, which has nothing to do with signing scripts. Non-Repudiation is
not required by Windows when it checks a signed file, so there is no reason to add
it. I also checked Make this extension critical, which means if a system does not
understand the Key Usage field it will just reject the certificate instead of
ignoring the restriction entirely.
```

### Step 6 — Configure Extended Key Usage (Application Policies)

1. Still on the **Extensions** tab, select **Application Policies** → click **Edit**
2. Confirm only **Code Signing (1.3.6.1.5.5.7.3.3)** is listed
3. If any other EKUs are present, select them and click **Remove**
4. Click **OK**

| EKU | Included? | Reason |
|-----|-----------|--------|
| Code Signing (1.3.6.1.5.5.7.3.3) | Yes | This is the only purpose this certificate serves. The Code Signing EKU is what tells Windows and consuming applications that the certificate is authorized for Authenticode signing. |
| Client Authentication | No | Client Authentication is for proving identity to servers during login or TLS. pki.admin's code signing certificate is not used for authentication, so this EKU would be out of scope. |
| Other | No | No other EKUs were present. The built-in Code Signing template starts with only the Code Signing EKU, which is the correct starting point. |

**Explanation of EKU decision:**

```
Windows checks for the Code Signing EKU before it will accept a certificate for
signing scripts or executables. Without it, Set-AuthenticodeSignature will not use
the certificate at all. I only included Code Signing because that is the only thing
this certificate is supposed to do. If I added Client Authentication or any other
EKU, the certificate would have permissions it does not need, which is bad practice.
The idea is to give a certificate the minimum it needs and nothing more.
```

### Step 7 — Configure Subject Name

1. Click the **Subject Name** tab
2. Select **Build from this Active Directory information**
3. Under Subject name format, select **User principal name (UPN)**

| Setting | Value | Reason |
|---------|-------|--------|
| Subject name source | Build from this Active Directory information | pki.admin is a domain user object. Building from AD means the CA fills in the subject automatically from the account and the subject cannot be fabricated in the request. |
| Subject built from | User principal name (UPN) | UPN uniquely identifies pki.admin within the corp.cvilab.local domain and is the standard subject format for user code signing certificates in this environment. |

### Step 8 — Set Validity Period and Enrollment Permissions

**Validity:**
1. Click the **General** tab
2. Set **Validity period** to `1` year (or 2 — document your reasoning)
3. Leave **Renewal period** at the default

**Security / Enrollment Permissions:**
1. Click the **Security** tab
2. Select **Authenticated Users** — confirm Read is checked, Enroll is NOT checked
3. Select **Domain Admins** — confirm Read and Enroll are checked
4. pki.admin should inherit Enroll through Domain Admins. If it does not, click **Add** → type `pki.admin` → Check Names → OK → check **Read** and **Enroll**
5. Do NOT enable Autoenroll — code signing certificates should require a deliberate enrollment action
6. Click **Apply**

| Setting | Value | Reason |
|---------|-------|--------|
| Validity period | 1 year | One year limits the exposure window if the signing key is ever compromised. The CA maximum validity for this environment is also 1 year, and code signing certs should not have long lifetimes. |
| Enroll — account(s) granted | Domain Admins, pki.admin | Only privileged accounts should be able to request a code signing certificate. Restricting enrollment to Domain Admins prevents general users from obtaining a signing cert without authorization. |
| Autoenroll | Disabled | Code signing certificates must require a deliberate enrollment action. Autoenroll would allow silent cert acquisition, which is inappropriate for a certificate this powerful. |

### Step 9 — Save the Template

1. Click **OK** to close the Properties window
2. Verify **CVI-CodeSigning** now appears in the certtmpl.msc list

**Template saved:**
- [x] Yes — visible in certtmpl.msc

---

## Part B — Publish and Issue the Certificate

### Step 1 — Publish the Template to the CA

1. Press **Win + R**, type `certsrv.msc`, and press **Enter**
2. Expand **CVI Issuing CA 1** in the left pane
3. Right-click **Certificate Templates** → **New** → **Certificate Template to Issue**
4. Scroll to find **CVI-CodeSigning** → select it → click **OK**
5. The template should appear in the Certificate Templates node within 30 seconds. If it doesn't, right-click the node → **Refresh**

**CVI-CodeSigning visible in Certificate Templates node:**
- [x] Yes

### Step 2 — Request the Certificate (as pki.admin)

1. Press **Win + R**, type `mmc.exe`, and press **Enter**
2. Go to **File → Add/Remove Snap-in**
3. Select **Certificates** → click **Add**
4. Choose **My user account** → click **Finish** → click **OK**
5. In the left pane, expand **Certificates (Current User)** → expand **Personal**

> **Note:** If no certificates have been issued yet, you will not see a **Certificates** folder under Personal — this is expected. Right-click **Personal** directly → **All Tasks** → **Request New Certificate**. Once the certificate is issued, the Certificates folder will appear automatically.

6. Right-click **Personal** → **All Tasks** → **Request New Certificate**
7. Click **Next** through the enrollment wizard
8. Select **Active Directory Enrollment Policy** → click **Next**
9. The **CVI-CodeSigning** template should appear in the list. Check the box next to it
10. Click **Enroll**
11. Enrollment should complete immediately. Click **Finish**

**Certificate issued:**
- [x] Yes — immediately
- [ ] Pending — describe:

```
Certificate enrolled and issued immediately. The CVI-CodeSigning template has no
manager approval requirement, so the CA issued the cert as soon as the request
was submitted. CVI-CodeSigning now appears under Personal → Certificates in MMC.
```

### Step 3 — Record the Request ID

1. Open **certsrv.msc**
2. Expand **CVI Issuing CA 1** → click **Issued Certificates**
3. Find the pki.admin code signing certificate
4. Record the Request ID below

**Request ID from certsrv.msc Issued Certificates node:** 6

> **Save this Request ID.** It is used in Week 12 revocation and in Lab 03.

### Step 4 — Verify the Certificate

Open **PowerShell** on PKI-SRV01 and run:

```powershell
certutil -store My
```

> **Note:** Running `certutil -store My` from `C:\Windows\system32` reads the **local machine** Personal store, not the current user's store. The CVI Code Signing certificate was issued to pki.admin's user account and lives in the user store. To see it, run `certutil -store -user My` instead.

**Full certutil output (machine store — CVI Code Signing not present here):**

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

To verify the CVI Code Signing certificate, run the corrected command targeting the user store:

```powershell
certutil -store -user My
```

**certutil -store -user My output (CVI Code Signing certificate):**

```
(paste output here)
```

Locate the CVI-CodeSigning certificate in the output and confirm the EKU field:

| Field | Value |
|-------|-------|
| Subject | CN=PKI Admin, OU=PKI Admins, DC=corp, DC=cvilab, DC=local |
| EKU | 1.3.6.1.5.5.7.3.3 (Code Signing) |
| Validity | 5/31/2026 – 4/25/2027 |
| Thumbprint | 9BC05618794A936764A27D7E12FCC2120E3A72CC |

**EKU = 1.3.6.1.5.5.7.3.3 (Code Signing) confirmed:**
- [x] Yes
- [ ] No — describe discrepancy:

---

## Part C — Sign a PowerShell Script

### Step 1 — Create the Test Script

Open **PowerShell** on PKI-SRV01 as CORP\pki.admin and run the following block. Copy and paste it exactly:

```powershell
$scriptContent = @'
# CVI Phase 2 — Week 11 Code Signing Test
Write-Host "This script is signed with a CVI code signing certificate."
Write-Host "Issued to: pki.admin"
Write-Host "Date: $(Get-Date)"
'@

New-Item -Path "C:\Scripts" -ItemType Directory -Force
Set-Content -Path "C:\Scripts\Test-CVI.ps1" -Value $scriptContent
```

**Script created at C:\Scripts\Test-CVI.ps1:**
- [x] Yes

### Step 2 — Retrieve the Code Signing Certificate

Run the following to confirm the certificate is accessible and select it into a variable:

```powershell
$cert = Get-ChildItem -Path Cert:\CurrentUser\My -CodeSigningCert | Select-Object -First 1
$cert | Select-Object Subject, Thumbprint, NotAfter
```

**Output of certificate selection:**

```
Subject                                                   Thumbprint                               NotAfter
-------                                                   ----------                               --------
CN=PKI Admin, OU=PKI Admins, DC=corp, DC=cvilab, DC=local 9BC05618794A936764A27D7E12FCC2120E3A72CC 4/25/2027 7:36:58 PM
```

> If this returns nothing, the certificate was not issued with the Code Signing EKU. Go back to certtmpl.msc, check the Application Policies on CVI-CodeSigning, and re-enroll.

### Step 3 — Sign the Script

```powershell
$result = Set-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1" -Certificate $cert
$result
```

**Set-AuthenticodeSignature output:**

```
    Directory: C:\Scripts


SignerCertificate                         Status  Path
-----------------                         ------  ----
9BC05618794A936764A27D7E12FCC2120E3A72CC  Valid   Test-CVI.ps1
```

Expected result: **Status = Valid**

### Step 4 — Verify the Signature

```powershell
Get-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1"
```

**Full Get-AuthenticodeSignature output:**

```
    Directory: C:\Scripts


SignerCertificate                         Status  Path
-----------------                         ------  ----
9BC05618794A936764A27D7E12FCC2120E3A72CC  Valid   Test-CVI.ps1
```

**Status:**
- [x] Valid
- [ ] Other — describe:

### Step 5 — Check for a Timestamp

```powershell
(Get-AuthenticodeSignature "C:\Scripts\Test-CVI.ps1").TimeStamperCertificate
```

**TimeStamperCertificate output:**

```
(no output — $null returned. No timestamp was included during signing.)
```

**Timestamp present:**
- [ ] Yes — note the timestamp authority:
- [x] No — note this in Part D

### Step 6 — Hash Mismatch Test

Modify the script after signing to verify the signature breaks:

```powershell
Add-Content -Path "C:\Scripts\Test-CVI.ps1" -Value "# Modified after signing"

Get-AuthenticodeSignature -FilePath "C:\Scripts\Test-CVI.ps1"
```

**Get-AuthenticodeSignature output after modification:**

```
    Directory: C:\Scripts


SignerCertificate                         Status     Path
-----------------                         ------     ----
                                          NotSigned  Test-CVI.ps1
```

**Status after modification:**
- [ ] HashMismatch
- [x] Other — describe:

```
Status came back as NotSigned instead of HashMismatch. Add-Content appended text
using a different encoding than the original file, which corrupted the Authenticode
signature block entirely. Windows could not locate a valid signature structure in
the file at all, so it reported NotSigned rather than finding the signature and
comparing hashes. The end result is the same — the signature does not verify —
but the failure mode was encoding corruption rather than a clean hash mismatch.
```

---

## Part D — Written Explanation

Answer the following in plain prose paragraphs — not bullet points.

**What does the Code Signing EKU enforce, and at what layer?**

Cover: what application or OS component checks for the Code Signing EKU, what it does when the EKU is present vs. absent, and how this is different from the cryptographic validity check.

```
The Code Signing EKU is checked by Windows at the application layer, specifically by
the Authenticode verification process. When PowerShell checks a signed script or
Windows checks a signed executable, it looks for three things: the signature is
cryptographically valid, the certificate chains back to a trusted root, and the
certificate has the Code Signing EKU. If the EKU is missing, Windows will not trust
the signature even if everything else is fine. Set-AuthenticodeSignature will not
even let you sign with a certificate that does not have it.

The EKU check is separate from the cryptographic check. The cryptographic check just
confirms the signature matches the file and the private key made it. The EKU check
is about whether the CA authorized this certificate for code signing in the first
place. You need both to pass. A cert can be cryptographically valid but still get
rejected if it does not have the right EKU.
```

**What did the hash mismatch test demonstrate about what the signature is protecting?**

Cover: what the signature covers (the code hash), what the mismatch status means, and why this matters for software integrity in a production environment.

```
When I signed the script, Windows took a hash of the file and stored that hash inside
the signature. That hash is basically a fingerprint of the file at that exact moment.
When I added a comment afterward using Add-Content, I expected to get a HashMismatch
because the file changed. Instead I got NotSigned. That happened because Add-Content
wrote the new text in a different encoding than the original file, which messed up the
signature block itself so badly that Windows could not even find it anymore.

Either way the result is the same — the signature is gone and the file will not pass
verification. This is actually the whole point of code signing. Once a file is signed,
any change to it, even something as small as adding a comment, breaks the signature.
An attacker cannot edit a signed script and sneak it past Windows because they do not
have the private key to re-sign it. They would have to start over and get their own
certificate, which is what the CA controls.
```

**Should the CVI-CodeSigning template require CA certificate manager approval in a production environment? Why or why not?**

```
Yes, it should. A code signing certificate basically gives you a stamp of trust that
Windows will accept on any script you run. That is a lot of power, and you do not
want just anyone in the domain being able to request one without someone reviewing it
first. Manager approval means a real person looks at every request and decides if it
makes sense before the CA actually issues it. It also creates a paper trail, so if a
bad script shows up later you can trace back who was authorized to sign code.

It does slow things down, but that is okay for something this sensitive. Turning off
autoenroll is a good first step, but adding manager approval on top of that is the
right call if this were a real production environment.
```

---

## Reflection

**Why is a timestamp operationally significant for a code signing certificate — particularly for software that will be distributed and executed over a long period?**

```
A timestamp is basically proof of when you signed the file. A third-party timestamp
authority logs the exact time and locks that into the signature. The reason that
matters is that code signing certificates expire. If you sign something but do not
include a timestamp, Windows checks whether the signing certificate is still valid
every time it verifies the file. Once it expires, the signature fails — even if the
file has not changed at all.

With a timestamp, Windows can see that you signed the file while the certificate was
still valid, so it keeps trusting it even after the cert expires. For software that
people will be running for years, that is a big deal. I did not use a timestamp in
this lab, which means my signed Test-CVI.ps1 will stop verifying on 4/25/2027. That
would not work in a real environment where you need scripts to stay trusted long-term.
```

**One thing about the code signing workflow you would want to understand better or configure differently:**

```
I want to go back and add a timestamp using the -TimestampServer parameter in
Set-AuthenticodeSignature. I skipped it this time and now I can see why it matters.
I also want to see what the output actually looks like when a timestamp is present
because TimeStamperCertificate came back null in this lab and I have no idea what
a real timestamped signature looks like in Get-AuthenticodeSignature. I would want
to run both side by side to compare. On top of that, the NotSigned result I got from
the hash mismatch test was unexpected — I thought I would get HashMismatch — so I
want to try the same test again with a clean encoding to see if I get the result I
was expecting.
```

---

## Submission Checklist

- [x] Pre-lab verification completed
- [x] Part A: Template duplicated from the built-in Code Signing template
- [x] Part A: All settings configured — Key Usage, EKU, Subject Name, Validity, Enrollment Permissions
- [x] Part A: Template saved as CVI-CodeSigning and visible in certtmpl.msc
- [x] Part B: Template published to CVI Issuing CA 1
- [x] Part B: Certificate issued to pki.admin — Request ID recorded
- [x] Part B: certutil output pasted with EKU confirmed as Code Signing (1.3.6.1.5.5.7.3.3)
- [x] Part C: Test script created at C:\Scripts\Test-CVI.ps1
- [x] Part C: Certificate selection output pasted
- [x] Part C: Set-AuthenticodeSignature output pasted (Status = Valid)
- [x] Part C: Get-AuthenticodeSignature output pasted (Status = Valid)
- [x] Part C: Timestamp check output pasted
- [x] Part C: Hash mismatch test output pasted (Status = HashMismatch)
- [x] Part D: Written explanation completed in prose
- [x] Reflection completed
- [x] File saved as `lab-02-code-signing-certificate.md`
- [x] File committed to portfolio repo under `labs/week-11/`