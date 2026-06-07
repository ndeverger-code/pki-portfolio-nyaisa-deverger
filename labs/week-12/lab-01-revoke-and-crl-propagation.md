# Lab 01: Revoke a Certificate and Observe CRL Propagation

**Student Name:** Nyaisa Deverger
**Date Completed:** June 7, 2026 
**Phase:** 2 | **Week:** 12  
**Submission Path:** `labs/week-12/lab-01-revoke-and-crl-propagation.md`

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as CORP\pki.admin, you are communicating with DC01 and the environment is ready. Proceed to Part A.

---

## Part A — Choose Your Certificate

You will revoke one certificate issued during Weeks 10 or 11. The choice of certificate matters — one is reserved for Week 15 and must not be revoked.

### Step 1 — Open the CA Console

1. Press **Win + R**, type `certsrv.msc`, and press **Enter**
2. The Certification Authority console opens
3. In the left pane, expand **CVI Issuing CA 1** → click **Issued Certificates**

### Step 2 — Identify a Certificate to Revoke

1. Browse the list to locate certificates issued in Weeks 10 or 11
2. Right-click the certificate you plan to revoke → **Properties**
3. Note the Common Name, Serial Number, and Certificate Template name
4. Close Properties — do not revoke yet

> **⚠️ Critical — Do NOT select svc.autoenroll:** The `svc.autoenroll` service account certificate is required for Week 15 autoenrollment testing. Revoking it now will break that lab environment. If you accidentally revoke it and the reason code was NOT Key Compromise, you can unrevoce it: go to **Revoked Certificates** → right-click the certificate → **Unrevoke**.

**Recommended revocation candidates (in order of preference):**

| Candidate | Template | Why It's a Good Choice |
|---|---|---|
| Code signing cert issued to CORP\pki.admin | CVI-CodeSigning | Best choice — no forward dependency in any remaining week |
| TLS certificate from Week 10 | CVI-WebServer | Acceptable |
| Any other issued cert that is not svc.autoenroll | Any | Acceptable |

### Step 3 — Record the Certificate Details

1. From an elevated PowerShell prompt, retrieve the serial number of your chosen certificate:

```powershell
certutil -view -restrict "CommonName=<cn-of-your-cert>" -out "RequestID,SerialNumber,CommonName,NotAfter"
```

**Certificate selected for revocation:**

```
Common Name / Subject: PKI Admin
```

**Template name:**

```
CVI Code Signing
```

**Serial number:**

```
Issued Request ID: 0x6
Serial Number: "44000000069be40163789196d9000000000006"
Issued Common Name: "PKI Admin"
Certificate Expiration Date: 4/25/2027 7:36 PM
CertUtil: -view command completed successfully.
```

**Why you selected this certificate (and confirmed it is not svc.autoenroll):**

```
Selected the CVI Code Signing certificate issued to CORP\pki.admin (PKI Admin). This certificate has no
forward dependency in any remaining lab week. The svc.autoenroll certificate (Request ID 5, Common Name
"Svc Autoenroll") was confirmed in the Issued Certificates list and was not selected.
```

---

## Part B — Revoke the Certificate

### Step 1 — Initiate the Revocation

1. In certsrv.msc → **Issued Certificates**, locate your chosen certificate in the list
2. Right-click the individual certificate row → **All Tasks → Revoke Certificate**

> **If the option is greyed out:** You right-clicked the Issued Certificates folder rather than an individual certificate row. Click the folder to open the list, then right-click a specific certificate row. Also confirm you are logged in as CORP\pki.admin, not a local account.

### Step 2 — Choose a Revocation Reason Code

1. The Revoke Certificate dialog opens with a dropdown for the reason code
2. Review the reason codes and choose the most appropriate one for your scenario:

| Reason Code | When to Use | Urgency | Reversible? |
|---|---|---|---|
| Key Compromise | Private key was exposed or stolen | Immediate — publish CRL now | No |
| CA Compromise | CA private key was exposed | Immediate + CA rebuild required | No |
| Affiliation Changed | Employee left or changed roles | Same-day | Yes |
| Superseded | Replacing with a new certificate | Next scheduled CRL cycle | Yes |
| Cessation of Operation | Service or system decommissioned | Next scheduled CRL cycle | Yes |

> **Why reason codes matter:** Key Compromise is the only code that requires immediate CRL publication regardless of the normal schedule. It also cannot be reversed — once a certificate is revoked for Key Compromise, it cannot be unrevoked. All other reason codes can be reversed from the Revoked Certificates view if revoked by mistake (except CA Compromise).

3. Select your reason code from the dropdown

### Step 3 — Confirm the Revocation

1. Click **Yes** to confirm
2. The certificate immediately moves from Issued Certificates to **Revoked Certificates**

### Step 4 — Record the Revocation Details

1. Click **Revoked Certificates** in the left pane
2. Locate your certificate and note the timestamp shown in the **Revocation Date** column

**Revocation reason code selected:**

| Reason Code | Selected? |
|---|---|
| Key Compromise | |
| CA Compromise | |
| Affiliation Changed | |
| Superseded | |
| Cessation of Operation | ✅ |

**Why you chose this reason code:**

```
Cessation of Operation was selected because the code-signing certificate issued to CORP\pki.admin
is no longer needed for the lab scenario — the purpose it was issued for has ended. This reason
code fits the scenario, is reversible if selected in error, and does not require an out-of-cycle
emergency CRL publication the way Key Compromise would.
```

**Certificate appears in Revoked Certificates view:**
- [x] Yes
- [ ] No — describe:

**Timestamp of revocation (from Revoked Certificates view):**

```
6/7/2026 10:15 AM
```

---

## Part C — Publish the CRL

After marking the certificate revoked, the revocation is internal to the CA. Relying parties have no way to know about it until a new CRL is published and distributed. This step makes the revocation visible to the outside world.

### Step 1 — Publish a New CRL from the CA Console

1. In certsrv.msc, right-click **CVI Issuing CA 1** — the CA name itself in the left pane, not a subfolder
2. Select **All Tasks → Publish**
3. In the Publish CRL dialog, select **New CRL** → click **OK**

> **Why "New CRL" and not "Delta CRL":** A New CRL is a complete, standalone revocation list. A Delta CRL only contains changes since the last Base CRL, so relying parties must have the Base CRL to use it. For a fresh lab environment where you want the revocation to be immediately verifiable without dependencies, publish a New CRL.

### Step 2 — Confirm Publication from PowerShell

1. In your elevated PowerShell prompt, run:

```powershell
certutil -CRL
```

Expected output:
```
CRL published successfully.
CertUtil: -CRL command completed successfully.
```

> **If you see "Access is denied":** You are not running from an elevated prompt, or you are not logged in as CORP\pki.admin. Right-click PowerShell → **Run as Administrator**. Confirm the session is CORP\pki.admin (not a local account) with `whoami`.

```
CertUtil: -CRL command completed successfully.
```

**CRL published without errors:**
- [x] Yes
- [ ] No — describe the error:

---

## Part D — Locate and Inspect the CRL

### Step 1 — Export the Revoked Certificate

1. In certsrv.msc → **Revoked Certificates**, right-click your certificate → **Export**
2. Save it as `revoked.cer` — you will use this file in Parts D and E

### Step 2 — Find the CRL Distribution Point URL in the Certificate

1. Run the following to extract the CDP URL from the certificate:

```powershell
certutil -dump revoked.cer | findstr "CDP http ldap"
```

The CDP URL in your environment will typically be in this format:
```
http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crl
```

There may also be an LDAP CDP entry for Active Directory-based distribution.

**CRL Distribution Point URL found in the certificate:**

```
No HTTP CDP is configured in this environment. The CA publishes the CRL to two locations:
  Local disk: C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl
  LDAP:       ldap:///CN=%7%8,CN=%2,CN=CDP,CN=Public Key Services,CN=Services,%6%10
The CRL was inspected directly from the local disk path above.
```

### Step 3 — Inspect the CRL on Disk

1. Dump the CRL and examine the key fields:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl"
```

> You can also download it via the HTTP CDP URL and inspect the downloaded file:
> ```powershell
> Invoke-WebRequest -Uri "<CDP-URL>" -OutFile downloaded.crl
> certutil -dump downloaded.crl
> ```

**Record the key fields from the CRL output:**

```
ThisUpdate: 6/7/2026 10:14 AM
```

```
NextUpdate: 6/14/2026 10:34 PM
```

> **ThisUpdate** is when this CRL was published — it should closely match the time you ran `certutil -CRL` in Part C. **NextUpdate** is when this CRL expires. Relying parties must download a fresh CRL before this time. With default settings, NextUpdate will be approximately one week from ThisUpdate.

### Step 4 — Confirm Your Revoked Certificate Appears in the CRL

1. Search the CRL for your certificate's serial number:

```powershell
certutil -dump "C:\Windows\System32\CertSrv\CertEnroll\CVI Issuing CA 1.crl" | findstr /i "<your-serial-number>"
```

```
  Serial Number: 44000000069be40163789196d9000000000006
   Revocation Date: 6/7/2026 10:15 AM
  Extensions: 1
    CRL Reason Code
        Cessation of Operation (5)
```

> **If the serial number does not appear:** You most likely published the CRL before completing the revocation, or you are looking at the wrong CRL file. Confirm the revocation appears in the Revoked Certificates view in certsrv.msc, then republish with `certutil -CRL`. Use the CDP URL from the certificate itself to confirm you are inspecting the correct CRL file.

**Your revoked certificate's serial number appears in the CRL:**
- [x] Yes
- [ ] No — describe what you see and what you suspect:

---

## Part E — Verify Revocation Status

### Step 1 — Run Certificate Verification

1. Run a full certificate verification against the revoked certificate:

```powershell
certutil -verify revoked.cer
```

### Step 2 — Clear the CRL Cache If Needed

1. If verification still shows the certificate as valid, the local CRL cache may be serving a stale copy. Clear it and retry:

```powershell
certutil -urlcache CRL delete
certutil -verify revoked.cer
```

**Expected output when revocation is confirmed:**
```
ERROR: Verifying leaf certificate revocation status returned The certificate is revoked.
0x80092010 (-2146885616 CRYPT_E_REVOKED)
CertUtil: -verify command failed. 0x80092010 (-2146885616 CRYPT_E_REVOKED)
```

> **If you see CRYPT_E_REVOCATION_OFFLINE instead:** The CRL is not accessible at the CDP URL — check network connectivity within the lab environment and confirm the CRL file is in `C:\Windows\System32\CertSrv\CertEnroll\`. Either output is acceptable for this lab if you include a written explanation of what you observed.

```
ChainContext.dwErrorStatus = CERT_TRUST_IS_REVOKED (0x4)

  Element.dwErrorStatus = CERT_TRUST_IS_REVOKED (0x4)
    CRL 0a:
    Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
    ThisUpdate: 6/7/2026 10:14 AM
    NextUpdate: 6/14/2026 10:34 PM

The certificate is revoked. 0x80092010 (-2146885616 CRYPT_E_REVOKED)
------------------------------------
Certificate is REVOKED
Leaf certificate is REVOKED (Reason=5)
CertUtil: -verify command completed successfully.
```

**certutil -verify reports the certificate as revoked:**
- [x] Yes — paste the relevant error text:
  `The certificate is revoked. 0x80092010 (-2146885616 CRYPT_E_REVOKED)` — Leaf certificate is REVOKED (Reason=5)
- [ ] No — describe what you see and what you tried:

---

## Part F — Lab Report

Answer all questions in your own words. Write in complete sentences.

**1. Walk through the two-step revocation workflow you performed. What happens at each step — and why is the CRL publication step not optional even when the certificate has already been revoked in the CA?**

```
The first step was marking the certificate as revoked inside the CA using certsrv.msc. This updates
the CA's internal database and moves the certificate from Issued Certificates to Revoked Certificates,
but nothing outside the CA knows about it yet. The second step was publishing a new CRL using
"All Tasks > Publish" on the Revoked Certificates folder, and then confirming with certutil -CRL.
This pushed the updated revocation list to the CertEnroll folder where relying parties can retrieve it.

The CRL publication step is not optional because the CA's internal database is private — no outside
system checks it directly. Relying parties like web browsers or code-signing validators only check
the CRL. If you skip publishing, the certificate still appears valid to anyone verifying it, even
though the CA considers it revoked. The revocation only becomes real to the outside world once the
new CRL is published and distributed.
```

**2. You chose a specific revocation reason code. What would have changed operationally if you had chosen Key Compromise instead — particularly regarding CRL publication timing and whether the revocation could be reversed?**

```
I chose Cessation of Operation because the certificate was no longer needed — it wasn't compromised,
just done being used. If I had chosen Key Compromise instead, two things would have changed. First,
Key Compromise signals that the private key may be in the wrong hands, so a new CRL would need to
be published immediately rather than waiting for the next scheduled cycle. This is treated as an
emergency. Second, Key Compromise is the one reason code that cannot be reversed. With Cessation
of Operation, I could go to the Revoked Certificates view and unrevoke the certificate if I made
a mistake. With Key Compromise, that option is gone permanently. Since the key was not actually
exposed in this lab, using Key Compromise would have been the wrong choice operationally.
```

**3. What does the NextUpdate field in the CRL tell a relying party? If a relying party tries to verify your revoked certificate after NextUpdate has passed and no new CRL has been published, what happens?**

```
The NextUpdate field tells relying parties when the current CRL expires and a new one is expected.
In this lab, NextUpdate was 6/14/2026 10:34 PM, meaning relying parties should download a fresh
CRL before that time. If a relying party tries to verify the certificate after that deadline and no
new CRL has been published, the behavior depends on the environment. In a hard-fail environment,
the verification will fail outright because the CRL is considered stale and untrusted. In a soft-fail
environment, the system may allow the verification to proceed since it cannot confirm the status either
way. Either way, the expired CRL is a problem — it means the CA has not kept up with its publishing
schedule, which is a reliability issue in production.
```

**4. A relying party in a soft-fail environment had already cached the CRL before you published the updated one in Part C. Would they know the certificate is revoked? Why or why not — and what mechanism exists to reduce this propagation lag?**

```
No, they would not know the certificate is revoked. When a relying party downloads a CRL, it caches
it locally until the NextUpdate time. If they cached the old CRL before I published the new one in
Part C, their cached copy does not contain the revoked serial number, so the certificate would still
appear valid to them. In a soft-fail environment, they would not even flag the missing entry as a
problem. The mechanism that exists to reduce this lag is the Delta CRL. A Delta CRL is a small,
frequently published list that only contains changes since the last Base CRL. Relying parties that
support Delta CRLs can check for recent changes more often without downloading the full CRL every
time. This significantly shortens the window between when a certificate is revoked and when relying
parties learn about it.
```

**5. What is one thing you would do differently or check additionally in a production environment that you did not need to do in this lab?**

```
In a production environment, I would verify that the CRL is actually reachable at its HTTP
distribution point before considering the revocation complete. In this lab, there was no HTTP CDP
configured — the CRL was only published to local disk and LDAP. In production, most relying parties
use an HTTP URL to download the CRL, so I would confirm the web server hosting the CRL is running,
the file is accessible from outside the CA server, and the URL in the certificate's CDP extension
actually resolves and returns the updated CRL. A revocation that cannot be downloaded by relying
parties is effectively the same as no revocation at all.
```

---

## Submission Checklist

- [x] Logged in as CORP\pki.admin (not a local account)
- [x] Certificate selected — NOT svc.autoenroll — serial number documented
- [x] Revocation reason code chosen with written rationale
- [x] Certificate confirmed in Revoked Certificates view with timestamp
- [x] CRL published — certutil -CRL output shows "CRL published successfully"
- [x] CRL Distribution Point URL identified from the certificate
- [x] CRL inspected: ThisUpdate and NextUpdate recorded
- [x] Revoked serial number confirmed present in the CRL
- [x] certutil -verify shows CRYPT_E_REVOKED (0x80092010) or CRYPT_E_REVOCATION_OFFLINE with explanation
- [x] All five lab report questions answered in complete sentences
- [x] Lab file committed to `labs/week-12/lab-01-revoke-and-crl-propagation.md`