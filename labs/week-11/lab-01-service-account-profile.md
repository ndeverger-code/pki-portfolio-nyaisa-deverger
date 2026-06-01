# Lab 01: Build a Certificate Profile for a Service Account

**Student Name:**  Nyaisa Deverger
**Date Completed:**  5/28/2026
**Phase:** 2 | **Week:** 11  
**Submission Path:** `labs/week-11/lab-01-service-account-profile.md`

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as **CORP\pki.admin**, you are communicating with DC01 and the environment is ready. Proceed to Part A.

```powershell
Get-Service -Name CertSvc
```

```
Status   Name               DisplayName
------   ----               -----------
Running  CertSvc            Active Directory Certificate Services
```

```powershell
certutil -ping
```

```
Connecting to PKI-SRV01.corp.cvilab.local\CVI Issuing CA 1 ...
Server "CVI Issuing CA 1" ICertRequest2 interface is alive (16ms)
CertUtil: -ping command completed successfully.
```

**CertSvc status:** Running
**CA responding (certutil -ping):** Yes

---

## Part A — Design the CVI-ServiceAccount Template

### Step 1 — Open the Certificate Templates Console

1. Press **Win + R**, type `certtmpl.msc`, and press **Enter**
2. The Certificate Templates console opens, showing all templates installed on this CA
3. Scroll through the list to get familiar with what already exists

### Step 2 — Duplicate the User Template

1. Scroll down to find the **User** template in the list
2. Right-click **User** → select **Duplicate Template**
3. A new template Properties window opens — this is your working copy


**Source template duplicated:** User

**Reason for choosing this source template:**

```
The svc.autoenroll account is an Active Directory user-type object, not a machine
account, so the User template is the correct baseline. The Computer template is
designed for machine accounts and includes Server Authentication EKU, which has no
place in a service account certificate. Starting from User also gives us the right
Subject Name behavior — it builds the subject from the AD user object — which is
exactly what we need for svc.autoenroll.
```

### Step 3 — Set Compatibility Settings

1. Click the **Compatibility** tab
2. Set **Certification Authority** to: `Windows Server 2012 R2`
3. Set **Certificate Recipient** to: `Windows 8.1 / Windows Server 2012 R2`
4. Click **OK** on any informational dialog that appears

**Compatibility settings selected:**
- Certification Authority: Windows Server 2012 R2
- Certificate Recipient: Windows 8.1 / Windows Server 2012 R2

### Step 4 — Set the Template Name (General Tab)

1. Click the **General** tab
2. Change **Template display name** to: `CVI Service Account`
3. The **Template name** (internal name, no spaces) will auto-fill as `CVI-ServiceAccount` — confirm this
4. Note the **Schema version** shown at the bottom

**General tab — Template names:**

| Field | Value |
|-------|-------|
| Template display name | CVI Service Account |
| Template name (internal) | CVIServiceAccount |
| Schema version | 4 |

### Step 5 — Configure Key Usage

1. Click the **Extensions** tab
2. In the Extensions list, select **Key Usage** → click **Edit**
3. In the Key Usage dialog:
   - Check **Digital Signature**
   - Uncheck **Key Encipherment** (if checked)
   - Uncheck **Data Encipherment** (if checked)
   - Uncheck **Non-Repudiation** (if checked)
   - Check **Make this extension critical**
4. Click **OK**

Document what you set and why:

| Key Usage | Included? | Reason |
|-----------|-----------|--------|
| Digital Signature | Yes | This certificate is used to prove who the service account is. Digital Signature lets the account sign something with its private key so the other side can verify it — that is how authentication works. |
| Key Encipherment | No | Key Encipherment is for encrypting keys during things like TLS handshakes. This service account is not doing any of that, so there is no reason to include it. |
| Data Encipherment | No | Data Encipherment is for directly encrypting files or data, like EFS does. svc.autoenroll is an authentication account, not a file encryption account, so this does not belong here. |
| Non-Repudiation | No | Non-Repudiation means the signature can be used as proof the person did something and cannot deny it. That is more of a human accountability thing — it does not make sense for an automated service account. |

**Explanation of Key Usage decisions:**

```
The only thing this certificate needs to do is let svc.autoenroll prove its identity,
which means Digital Signature is the only key usage it needs. When the service account
authenticates to something, it uses its private key to sign a challenge and the other
side checks the signature with the public key. That is what Digital Signature covers.

I left out Key Encipherment because that is for encrypting keys during things like TLS
handshakes, which this account is not doing. I left out Data Encipherment because that
is for encrypting actual data like files, which also is not what svc.autoenroll does.
Non-Repudiation got removed because that is about proving a specific person made a
decision and cannot take it back — it does not apply to an automated service account
that runs in the background. I also checked "Make this extension critical" so that any
system that does not recognize the Key Usage field will reject the cert instead of
just ignoring the restriction.
```

### Step 6 — Configure Extended Key Usage (Application Policies)

1. Still on the **Extensions** tab, select **Application Policies** → click **Edit**
2. The User template comes with several EKUs pre-populated (Client Authentication, Encrypting File System, Secure Email). You need to remove all but one:
   - Select **Encrypting File System** → click **Remove**
   - Select **Secure Email** → click **Remove**
   - Leave **Client Authentication** (1.3.6.1.5.5.7.3.2) — this is the only EKU needed
3. Click **OK**

Document your EKU decisions:

| EKU | Included? | OID | Reason |
|-----|-----------|-----|--------|
| Client Authentication | Yes | 1.3.6.1.5.5.7.3.2 | This is the only thing svc.autoenroll needs to do — authenticate itself to other systems. Client Authentication is what tells a server that this certificate can be used to log in or prove identity. |
| Server Authentication | No | 1.3.6.1.5.5.7.3.1 | Server Authentication is for things like web servers proving who they are over TLS. This is a service account, not a server, so this EKU does not belong here. |
| Code Signing | No | 1.3.6.1.5.5.7.3.3 | Code Signing is for signing software so people know it has not been tampered with. svc.autoenroll is not signing any code. |
| Secure Email | No | 1.3.6.1.5.5.7.3.4 | Secure Email (S/MIME) is for signing and encrypting emails. This account does not send email, so this EKU was removed. |

**Explanation of EKU decisions:**

```
The only thing this certificate needs to do is let svc.autoenroll authenticate to
other systems, so Client Authentication is the only EKU it needs. The User template
comes with Encrypting File System and Secure Email already included, but those are
for completely different jobs. EFS is for file encryption, and S/MIME is for email
— neither of those apply to a service account that just needs to log in somewhere.

Leaving extra EKUs in would be bad practice because it gives the certificate more
powers than it actually needs. If the account or certificate ever got compromised,
an attacker could potentially use those extra capabilities in ways that were not
intended. Keeping it to just Client Authentication follows the principle of least
privilege — the cert does exactly one thing and nothing more.
```

### Step 7 — Configure Subject Name

1. Click the **Subject Name** tab
2. Select **Build from this Active Directory information**
3. Under **Subject name format**, select **User principal name (UPN)** from the dropdown
4. Uncheck any other options under "Include this information in alternate subject name" that are not needed

Document your settings:

| Setting | Value Selected | Reason |
|---------|---------------|--------|
| Subject name format | Common name | UPN is not available as a primary subject name format at this schema version. Common name pulls the account's CN from Active Directory, which uniquely identifies svc.autoenroll. |
| Include this information in alternate subject name | User principal name (UPN) | The UPN (svc.autoenroll@corp.cvilab.local) is included in the SAN so Windows can use it for authentication lookups. E-mail name, DNS name, and SPN were left unchecked since they are not relevant for this account. |

**Explanation of Subject Name decision:**

```
I chose "Build from this Active Directory information" instead of "Supply in the
request" because svc.autoenroll is a known account in AD. Building from AD means
the CA automatically fills in the subject name from the account's information, so
there is no chance of someone requesting a cert with a fake or wrong name. Supply
in the request would let anyone type whatever they want as the subject, which opens
up the door for misuse.

The primary subject format is set to Common name, which pulls the CN field from
the AD account object. The UPN is checked under alternate subject name so Windows
can match the certificate back to the correct account during authentication. Email
name and DNS name were left unchecked because this is an authentication-only
certificate for a service account — those fields serve no purpose here.
```

### Step 8 — Set Validity Period

1. Click the **General** tab (you may already be on it)
2. Find the **Validity period** and **Renewal period** fields
3. Set **Validity period** to `1` year (or 2 years — document your reasoning)
4. Leave **Renewal period** at the default (6 weeks)

> **Note:** The CA itself has a maximum validity setting. If you set 5 years here but certs issue for only 2, the CA's max is overriding your template. You can check with: `certutil -getreg ca\ValidityPeriod`

| Setting | Value | Reason |
|---------|-------|--------|
| Validity period | 1 year | One year balances security with operational overhead. It is short enough to limit the exposure window if svc.autoenroll is compromised, and aligns with the CA maximum validity. Autoenrollment handles renewal automatically, so annual rotation is low-friction. |
| Renewal period | 6 weeks | Default setting. Six weeks gives autoenrollment sufficient lead time to renew before expiration without creating gaps in service. |

**Explanation of validity period decision:**

```
A one-year validity period was chosen because it limits the exposure window if the
service account or its private key is ever compromised, while remaining manageable
in an environment with autoenrollment configured. Since the CVI-ServiceAccount
template has autoenroll permissions for svc.autoenroll, renewal happens automatically
before expiration — making the short lifecycle essentially invisible operationally.
Setting a longer period (e.g., 5 years) would reduce administrative overhead but
increase risk: a compromised certificate would remain valid for years. The CA maximum
validity is also 1 year for this lab environment, so setting more than 1 year here
would be silently capped by the CA anyway.
```

### Step 9 — Set Enrollment Permissions (Security Tab)

1. Click the **Security** tab
2. You will see **Authenticated Users** and **Domain Admins** already listed
3. Add svc.autoenroll manually:
   - Click **Add**
   - In the object picker, type `svc.autoenroll` and click **Check Names**
   - Confirm the account resolves to `CORP\svc.autoenroll`, then click **OK**
   - With **svc.autoenroll** selected in the list, check **Read**, **Enroll**, and **Autoenroll**
4. Click **Apply**
5. Review the permissions on all three entries and document them below

| Group / Account | Read | Enroll | Autoenroll | Reason |
|-----------------|------|--------|------------|--------|
| Authenticated Users | Yes | No | No | Authenticated Users can read the template so their machines can see it exists, but they cannot enroll. This keeps the template visible without opening enrollment to everyone in the domain. |
| CORP\svc.autoenroll | Yes | Yes | Yes | svc.autoenroll is the only account that should be getting this certificate, so it needs all three permissions. Read lets it see the template, Enroll lets it request a cert, and Autoenroll lets it renew automatically. |
| Domain Admins | Yes | Yes | Yes | Domain Admins have full control over templates by default. They can enroll and autoenroll as needed for administrative purposes. |
| pki.admin | Yes | Yes | Yes | The PKI admin account was present and retains its default elevated permissions on this template. |
| Domain Users | Yes | No | No | Domain Users get Read only — same reasoning as Authenticated Users. They can see the template but cannot request a certificate from it. |
| Enterprise Admins | Yes | Yes | Yes | Enterprise Admins have full control over PKI infrastructure by default. |

**Explanation of enrollment permission decisions:**

```
The most important permission decision here was giving Enroll and Autoenroll only to
svc.autoenroll and not to Authenticated Users or Domain Users. If Authenticated Users
had Enroll, then any user or computer in the domain could request a certificate from
this template. That would mean people could get a certificate that authenticates as
a service account context, which is a security problem. Restricting Enroll to just
svc.autoenroll makes sure only that specific account can get this type of certificate.

Autoenroll is checked for svc.autoenroll so the certificate can renew itself
automatically before it expires. Without Autoenroll, someone would have to manually
request a new certificate every year, which is easy to forget and could cause service
outages. Having Read on Authenticated Users is fine because that just lets systems
discover the template exists — it does not let them actually request a cert from it.
```

### Step 10 — Save the Template

1. Click **OK** to close the Properties window and save the template
2. Verify **CVI-ServiceAccount** now appears in the certtmpl.msc list

**Template saved and visible in certtmpl.msc:**
- [x] Yes — CVI Service Account appears in the list with Schema Version 4 and Intended Purpose: Client Authentication

---

## Part B — Publish the Template and Issue the Certificate

### Step 1 — Publish the Template to the CA

1. Press **Win + R**, type `certsrv.msc`, and press **Enter**
2. Expand **CVI Issuing CA 1** in the left pane
3. Right-click **Certificate Templates** → **New** → **Certificate Template to Issue**
4. In the Enable Certificate Templates dialog, scroll to find **CVI-ServiceAccount**
5. Select it → click **OK**
6. The template should appear in the Certificate Templates node within 30 seconds. If it doesn't, right-click the node → **Refresh**

**CVI-ServiceAccount visible in Certificate Templates node:**
- [x] Yes
- [ ] No — describe what happened:

```
The template was already published to the CA when checked — it appeared in the
Certificate Templates node as 'CVI Service Account' with Intended Purpose:
Client Authentication. The "New → Certificate Template to Issue" dialog did not
show it because it was already added. Get-CATemplate confirmed it was present.
```

---

### Step 2 — Request the Certificate for svc.autoenroll

> **Important:** svc.autoenroll is an AD user account, not a Windows service. The Certificates snap-in "Service account" option lists Windows system services only — svc.autoenroll will not appear there. Instead, use `runas` to open MMC running as the svc.autoenroll account.

1. Open **Command Prompt** or **PowerShell** as Administrator
2. Run the following command:
   ```
   runas /user:CORP\svc.autoenroll mmc.exe
   ```
3. Enter the **svc.autoenroll password** when prompted
4. A new MMC window opens — this window is running as svc.autoenroll
5. In the new MMC window: **File → Add/Remove Snap-in**
6. Select **Certificates** → click **Add**
7. Choose **My user account** → click **Finish** → click **OK**
8. Expand **Certificates (Current User)** → **Personal**

> **Note:** If no certificates have been issued yet for this account, you will not see a **Certificates** folder under Personal — this is expected. Right-click **Personal** directly → **All Tasks** → **Request New Certificate**. Once the certificate is issued, the Certificates folder will appear automatically.

9. Right-click **Personal** → **All Tasks** → **Request New Certificate**
10. Click **Next** through the enrollment wizard
11. Select **Active Directory Enrollment Policy** → click **Next**
12. The CVI-ServiceAccount template should appear in the list. Check the box next to it
13. Click **Enroll**
14. Enrollment should complete immediately (Status: Succeeded). Click **Finish**

**Enrollment wizard — enrollment policy selected:**

```
Active Directory Enrollment Policy
```

**Templates visible in the wizard:**

```
- 'CVI Service Account'    — STATUS: Available
- Authenticated Session    — STATUS: Available
- Basic EFS                — STATUS: Available
- User                     — STATUS: Available
- Administrator            — STATUS: Unavailable (insufficient permissions)
```

**CVI-ServiceAccount visible in wizard:**
- [x] Yes
- [ ] No — troubleshooting steps taken:

**Certificate request submitted:**
- [x] Yes — issued immediately
- [ ] Yes — pending manager approval (describe resolution):
- [ ] No — error encountered:

```
N/A — certificate issued without delay.
```

---

## Part C — Verify the Issued Certificate

### Step 1 — Inspect the Certificate with certutil

Open **PowerShell** on PKI-SRV01 as CORP\pki.admin and run:

```powershell
certutil -store My
```

> If the certificate was enrolled via the runas MMC session, it will be in the svc.autoenroll user's personal store, not the pki.admin store. To view it, you can either re-open the runas MMC window, or check the CA's Issued Certificates node (Step 2 below) to confirm issuance.

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

> Note: certutil output captured via MMC Certificate Details tab (runas session store not directly accessible from pki.admin context).

**From the certutil output — record the following:**

| Field | Value |
|-------|-------|
| Subject | CN=Svc Autoenroll (Service Account, corp.cvilab.local) |
| Issuer | CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local |
| Serial Number | 44000000059271f2d220b1aef8000000000005 |
| Key Usage | Digital Signature, Key Encipherment (a0) |
| Enhanced Key Usage (EKU) | Client Authentication (1.3.6.1.5.5.7.3.2) |
| Validity: Not Before | Sunday, May 31, 2026 5:37 PM |
| Validity: Not After | Sunday, April 25, 2027 7:36 PM |
| Thumbprint | 2d3f56c65ba9dae586111f826f9e5ab08a8cb6d8 |

### Step 2 — Confirm in certsrv.msc and Record the Request ID

1. Open **certsrv.msc** (Press Win + R → type `certsrv.msc`)
2. Expand **CVI Issuing CA 1** → click **Issued Certificates**
3. Find the svc.autoenroll certificate in the list (sort by Request ID or Requester Name)
4. Double-click it to open and confirm it shows the CVI-ServiceAccount template
5. Record the values below

**Certificate visible in Issued Certificates node:**
- [x] Yes

**Record from the Issued Certificates node:**

| Column | Value |
|--------|-------|
| Request ID | 5 |
| Requester Name | CORP\svc.autoenroll |
| Certificate Template | 'CVI Service Account' |
| Issued Common Name | Svc Autoenroll |
| Certificate Expiration Date | 4/25/2027 7:36 PM |

> **Save this Request ID.** You will use it in Week 12 to revoke this certificate, and in Lab 03 for the comparison exercise.

---

## Part D — Written Explanation

Answer the following questions in plain prose paragraphs — not bullet points. Aim for 2–3 paragraphs total across the two questions.

**What makes a service account certificate different from a user certificate? Address the following:**

1. Key difference in EKU — what does a user certificate include that the service account certificate should not, and why?
2. Subject Name source — both use "Build from AD," but what is different about the identity being represented?
3. Enrollment — who requests a user certificate vs. who requests a service account certificate, and why does this matter?

```
A service account certificate and a regular user certificate are both issued by the
same CA, but they are built for very different purposes. The biggest difference is
the EKU. A normal user certificate comes with Encrypting File System and Secure
Email because actual people need those things — they encrypt files and send signed
emails. A service account certificate should not have any of that because
svc.autoenroll is not a person. It is an automated account that just needs to
prove its identity to other systems. Leaving those extra EKUs on would give the
certificate more power than it needs, and that is always a bad idea from a
security standpoint.

The subject name is another difference, even though both use "Build from Active
Directory." When a user certificate is built from AD, it pulls from a real
person's account — their name, email address, and UPN. When a service account
certificate is built from AD, it pulls from an account that belongs to an
automated process, not a human. The name on the certificate is svc.autoenroll,
which just tells other systems which service account this is. There is no real
person attached to it.

Enrollment is different too. A regular user can just open MMC while logged into
their own account and request a certificate through the wizard. That does not
work for a service account because you cannot just log in as svc.autoenroll
normally — it is not set up for interactive logins. Instead you have to use
runas to open MMC in the context of that account, or set up autoenrollment to
handle it automatically. This matters because if you do not plan for it ahead
of time, the service account will never get a certificate and the whole setup
falls apart.
```

**What are the operational risks of relying on password authentication for service accounts instead of certificate-based authentication?**

```
One big risk of using passwords for service accounts is that passwords can get
stolen or leaked and you might not find out for a long time. Service account
passwords usually never get changed because updating them means going into every
system that uses that account and changing the password there too. So if someone
gets that password, they could be using it quietly for months or even years before
anyone catches it. With a certificate, the private key never gets sent anywhere,
so there is nothing to steal in the same way.

The other risk is that passwords have to be stored somewhere. Whether it is a
config file, a script, or some kind of secrets manager, there is always a place
where the password lives in a readable form. If that file gets copied to the wrong
place, checked into source control by accident, or someone just screenshots it,
the credential is out. Certificate-based authentication avoids that whole problem
because there is no password to store anywhere — the certificate and the private
key do the work through cryptography, and the key never has to leave the machine.
```

---

## Reflection

**One thing about the CVI-ServiceAccount template design that was a non-obvious decision:**

```
The most non-obvious part was the Subject Name tab. I went in expecting to just
pick "User principal name (UPN)" from the dropdown for the subject name format,
but it was not there. Turns out UPN is not an option for the primary subject name
format at schema version 4 — it only shows up under the alternate subject name
section. So the right setup was to use Common name as the primary format and
check UPN under alternate subject name, which is actually what Windows checks
when it matches a certificate to an account anyway. I would not have known that
without working through it in the lab.
```

**What would you change about this template if this were a production environment rather than a lab?**

```
If this were production, the first thing I would change is adding a manager
approval step so someone has to review and sign off before a certificate gets
issued to this account. That creates an audit trail and prevents the cert from
being issued without anyone knowing. I would also shorten the validity period to
six months instead of a year — autoenrollment handles the renewals automatically
so the shorter period would not be a burden, and it limits how long a compromised
cert could be used. Lastly, I would set up some kind of expiration alert so that
if autoenrollment ever fails, there is a warning before the cert expires and
causes a service outage.
```

---

## Submission Checklist

- [x] Pre-lab verification completed
- [x] Part A: Template duplicated from the User template
- [x] Part A: All five design decisions (Key Usage, EKU, Subject Name, Validity, Security) documented with rationale
- [x] Part A: Template saved as CVI-ServiceAccount and visible in certtmpl.msc
- [x] Part B: Template published to CVI Issuing CA 1
- [x] Part B: Certificate requested via runas /user:CORP\svc.autoenroll mmc.exe
- [x] Part B: Certificate issued — enrollment outcome documented
- [x] Part C: certutil output pasted and key fields extracted into table
- [x] Part C: Request ID recorded from certsrv.msc Issued Certificates node
- [x] Part D: Written explanation completed in prose
- [x] Reflection section completed
- [x] File saved as `lab-01-service-account-profile.md`
- [x ] File committed to portfolio repo under `labs/week-11/`