# Lab 02: Configure and Test the OCSP Responder

**Student Name:** Nyaisa Deverger
**Date Completed:** June 7, 2026 
**Phase:** 2 | **Week:** 12  
**Submission Path:** `labs/week-12/lab-02-ocsp-responder.md`

---

## Prerequisites

Complete Lab 01 before starting Lab 02. You need:
- A **revoked certificate** from Lab 01 (for OCSP Revoked status testing)
- A **valid, non-revoked certificate** (your TLS cert from Week 10 or another cert from Week 11 that you did not revoke)

---

## Known Environment Note

The lab OVA was built without HTTP-based AIA and CDP extensions on the CA. This means certificates issued before Part 0 will show only LDAP URLs in their AIA and CDP extensions — no `http://pki-srv01.corp.cvilab.local/ocsp` entry will appear. This is not something you caused or misconfigured. Part 0 below corrects this before you proceed.

---

## Part 0 — Configure HTTP AIA and CDP Extensions on the CA

> **Why this matters:** The Online Responder requires an HTTP-accessible OCSP URL in the AIA extension of issued certificates. Without it, relying parties cannot locate the OCSP responder, and the Online Responder snap-in will report stale or unavailable CRL data. This part adds the correct HTTP entries to the CA and re-issues your test certificates so they reflect the correct URLs.

### Step 1 — Add HTTP CDP and AIA Extensions

1. On **PKI-SRV01**, open an **elevated PowerShell prompt**
2. Run the following commands exactly as shown:

```powershell
# Set HTTP CDP (CRL Distribution Point)
certutil -setreg CA\CRLPublicationURLs "65:C:\Windows\system32\CertSrv\CertEnroll\%3%8%9.crl\n6:http://pki-srv01.corp.cvilab.local/CertEnroll/%3%8%9.crl"

# Set HTTP AIA (CA certificate download + OCSP)
certutil -setreg CA\CACertPublicationURLs "1:C:\Windows\system32\CertSrv\CertEnroll\%1_%3%4.crt\n2:http://pki-srv01.corp.cvilab.local/CertEnroll/%1_%3%4.crt\n32:http://pki-srv01.corp.cvilab.local/ocsp"
```

3. Restart the Certificate Services and publish a fresh CRL:

```powershell
net stop certsvc
net start certsvc
certutil -CRL
```

Expected output from `certutil -CRL`:
```
CRL published.
```

> **What the flags mean:** In the CDP string, `65` = publish to the local file system and include in certificates. `6` = publish to the HTTP URL and include in certificates. In the AIA string, `1` = local file, `2` = HTTP download URL for the CA cert, `32` = OCSP URL.

**HTTP AIA and CDP configured and CertSvc restarted:**
- [x] Yes
- [ ] No — describe the error:

```
CertUtil: -CRL command completed successfully.
```

### Step 2 — Re-Issue Your Test Certificates

Because the CA extension changes only apply to newly issued certificates, you need to re-issue the two certificates you will use in Parts D and E: one valid cert and one that you will revoke.

1. In **certsrv.msc** on PKI-SRV01, issue a new certificate using any available template (WebServer or a template from a prior lab)
2. Export it as a `.cer` file — this will be your **valid test certificate** for Part D
3. Issue a second certificate from the same template
4. Export it as a second `.cer` file — you will revoke this one in Step 3

> If you have an existing valid certificate from Week 10 or 11 and prefer to keep it, you may re-use it — but note that it will not contain the HTTP OCSP URL in its AIA extension. For Part D and E to work as documented, use the newly issued certificates.

### Step 3 — Revoke the Second Certificate

1. In **certsrv.msc**, locate the second certificate you just issued under **Issued Certificates**
2. Right-click it → **All Tasks → Revoke Certificate**
3. Select reason **Key Compromise** → click **Yes**
4. Publish a new CRL immediately:

```powershell
certutil -CRL
```

5. This revoked certificate will be your **revoked test certificate** for Part D — it replaces the Lab 01 revoked certificate for the purposes of OCSP testing

### Step 4 — Confirm the HTTP OCSP URL Appears in the New Certificate

1. Run a dump of your newly issued valid certificate:

```powershell
certutil -dump <valid-certificate.cer> | findstr /i "ocsp"
```

Expected output should include:
```
OCSP Authority Info Access
    URL=http://pki-srv01.corp.cvilab.local/ocsp
```

**HTTP OCSP URL confirmed present in newly issued certificate:**
- [x] Yes
- [ ] No — do not proceed. Check that `certutil -CRL` ran without error and that CertSvc restarted successfully. Re-issue the certificate and try again.

```
URL=http://pki-srv01.corp.cvilab.local/ocsp
```

---

## Pre-Lab Verification

If you can log into PKI-SRV01 as CORP\pki.admin, you are communicating with DC01 and the environment is ready. Proceed to Part A.

---

## Part A — Install the Online Responder Role

### Step 1 — Open Add Roles and Features

1. On PKI-SRV01, open **Server Manager**
2. Click **Manage → Add Roles and Features**

### Step 2 — Select Installation Type

1. On the Installation Type page, select **Role-based or feature-based installation**
2. Click **Next**

### Step 3 — Confirm Server Selection

1. On the Server Selection page, confirm **PKI-SRV01** is selected
2. Click **Next**

### Step 4 — Add the Online Responder Role

1. On the Server Roles page, expand **Active Directory Certificate Services**
2. Check **Online Responder**
3. When prompted to add required features, click **Add Features**
4. Click **Next**

### Step 5 — Complete the Installation

1. Continue through the remaining pages without changes
2. Click **Install**
3. Wait for the installation to complete before continuing — do not close Server Manager or proceed to Part B until the progress bar finishes

### Step 6 — Verify the Online Responder Service

1. In your elevated PowerShell prompt, confirm the service installed and is running:

```powershell
Get-Service -Name OCSPSvc
```

Expected output:
```
Status   Name     DisplayName
------   ----     -----------
Running  OCSPSvc  Online Responder Service
```

> **If OCSPSvc does not appear:** The role installation may not have completed. Check Server Manager → Local Server for any pending configuration tasks. A server restart may be required on some configurations.

**Installation completed successfully:**
- [x] Yes
- [ ] No — describe the error:

```
Status   Name     DisplayName
------   ----     -----------
Running  OCSPSvc  Online Responder Service
```

---

## Part B — Configure the Revocation Configuration

### Step 1 — Open the Online Responder Snap-in

1. Navigate to:
```
Start → Administrative Tools → Online Responder
```

If Online Responder is not visible in Administrative Tools:
1. Press **Win + R**, type `mmc.exe`, press **Enter**
2. Go to **File → Add/Remove Snap-ins**
3. Select **Online Responder** → click **Add** → click **OK**

### Step 2 — Start the Add Revocation Configuration Wizard

1. In the Online Responder snap-in, right-click **Revocation Configuration**
2. Select **Add Revocation Configuration**

### Step 3 — Work Through the Wizard

1. In the wizard, configure each page as follows:

| Wizard Page | Setting | Value |
|---|---|---|
| Name | Descriptive name for this configuration | `CVI Issuing CA 1 Revocation` (or similar) |
| CA Certificate Source | How to find the CA certificate | Select a certificate for an Existing enterprise CA |
| Browse | Select the CA | CVI Issuing CA 1 |
| Signing Certificate | How to obtain the signing cert | Automatically select a signing certificate (leave default) |

2. Click **Next** through each page, then click **Finish**

> **Why "Automatically select a signing certificate":** The Online Responder will automatically request an OCSP Response Signing certificate from the CA using the built-in OCSP Response Signing template. This is the correct behavior — do not select a certificate manually unless instructed.

### Step 4 — Read the Status Indicator

1. After the wizard completes, the Online Responder snap-in shows a status indicator for your revocation configuration
2. Wait up to 60 seconds, then press **F5** to refresh

| Status Indicator | Meaning | What to Do |
|---|---|---|
| Green | Operational — OCSP signing certificate has been issued and the responder is active | No action needed |
| Yellow | Configuration pending — signing cert may not have been issued yet | Wait 30–60 seconds and press F5 to refresh. If still yellow after 2 minutes, check Part C. |
| Red | Configuration failed — signing cert missing or CA unreachable | Confirm CertSvc is running. Check certsrv.msc → Certificate Templates and confirm the OCSP Response Signing template is published. |

**Revocation configuration name entered:**

```
CVI Issuing CA 1 Revocation
```

**CA certificate found and selected:**
- [x] Yes
- [ ] No — describe:

**Status indicator after configuration:**
- [x] Green (operational)
- [ ] Yellow (warning — describe below)
- [ ] Red (error — describe below)

```
CVI Issuing CA 1 Revocation    Working
Signing Certificate: Ok
Revocation Provider Status: The revocation provider is successfully using the current configuration.
```

---

## Part C — Verify the OCSP Signing Certificate

When you created the revocation configuration, the Online Responder automatically requested an OCSP signing certificate from CVI Issuing CA 1. This certificate is what the Online Responder uses to sign every OCSP response it issues. You need to confirm it has the correct properties.

### Step 1 — List the Personal Store Certificates

1. In an elevated PowerShell prompt, list all certificates in the machine Personal store:

```powershell
certutil -store My
```

2. Locate the certificate with `OCSP Signing` in its Enhanced Key Usage section

```
Serial Number: 4400000009ed6babc1c3b90117000000000009
Issuer: CN=CVI Issuing CA 1, DC=corp, DC=cvilab, DC=local
NotBefore: 6/7/2026 12:47 PM
NotAfter: 6/21/2026 12:47 PM
Subject: CN=PKI-SRV01.corp.cvilab.local
Template: OCSPResponseSigning, OCSP Response Signing
Enhanced Key Usage: OCSP Signing (1.3.6.1.5.5.7.3.9)
Key Usage: Digital Signature (80)
OCSP No Revocation Checking: 1.3.6.1.5.5.7.48.1.5 (present)
Cert Hash(sha1): 2a923bc3d158ed655551d11ba7f6e558bb29fe20
```

### Step 2 — Inspect the OCSP Signing Certificate

1. Run a targeted inspection using the certificate's subject name or thumbprint from Step 1:

```powershell
certutil -store My "<OCSP signing cert subject or thumbprint>"
```

### Step 3 — Verify the Required Properties

1. Confirm the following properties are present in the certutil output:

| Property | Expected Value | Observed Value |
|---|---|---|
| Extended Key Usage | OCSP Signing — OID 1.3.6.1.5.5.7.3.9 | OCSP Signing (1.3.6.1.5.5.7.3.9) ✅ |
| id-pkix-ocsp-nocheck extension | Present — OID 1.3.6.1.5.5.7.48.1.5 | Present (OCSP No Revocation Checking) ✅ |
| Key Usage | Digital Signature | Digital Signature (80) ✅ |
| Issuer | CVI Issuing CA 1 | CN=CVI Issuing CA 1 ✅ |
| Validity (NotAfter) | In the future | 6/21/2026 12:47 PM ✅ |

**Why these properties are required:**

| Property | Why Required |
|---|---|
| OCSP Signing EKU (1.3.6.1.5.5.7.3.9) | Authorizes this key specifically for signing OCSP responses. Without it, relying parties reject the response signature. |
| id-pkix-ocsp-nocheck (1.3.6.1.5.5.7.48.1.5) | Tells relying parties NOT to check the revocation status of the OCSP signing cert itself. Without it, checking an OCSP response would require another OCSP query — creating an infinite circular dependency. |
| Short validity (auto-renewed) | The Online Responder auto-renews the signing cert before expiry. Short validity limits exposure if the signing key is ever compromised. |
| Issued by CVI Issuing CA 1 | The signing cert must chain to the same CA as the certificates it is responding about. Clients validate this chain. |

> **If the OCSP Signing EKU is missing:** The wrong template was used during revocation configuration setup. Delete the revocation configuration in the snap-in and recreate it — the wizard should auto-select the correct OCSP Response Signing template when the Online Responder role is properly configured.

**OCSP signing certificate subject:**

```
CN=PKI-SRV01.corp.cvilab.local
Serial: 4400000009ed6babc1c3b90117000000000009
Template: OCSP Response Signing
NotAfter: 6/21/2026 12:47 PM
```

**All expected properties present:**
- [x] Yes
- [ ] No — describe what is missing:

---

## Part D — Test the OCSP Responder

### Step 1 — Find the OCSP URL in the AIA Extension

1. Export a certificate to a `.cer` file if needed (from certsrv.msc → Issued or Revoked Certificates → right-click → Export), then run:

```powershell
certutil -dump <certificate.cer> | findstr /i "OCSP Authority"
```

The OCSP URL format in your environment is typically:
```
http://pki-srv01.corp.cvilab.local/ocsp
```

**OCSP URL found in the AIA extension:**

```
http://pki-srv01.corp.cvilab.local/ocsp
```

### Step 2 — Test a Valid Certificate

1. Use a certificate that was NOT revoked in Lab 01 — your TLS certificate from Week 10, or any other valid certificate
2. Run:

```powershell
certutil -URL <valid-certificate.cer>
```

3. In the URL Retrieval Tool window:
   - Select **OCSP** from the dropdown
   - Click **Retrieve**

Expected result: **OK (0) — Certificate is Good**

**OCSP response for the VALID certificate:**
- [x] Good
- [ ] Revoked (unexpected — investigate)
- [ ] Unknown
- [ ] Connection timeout / error

```
Status: Verified
Type: OCSP
Url: [0.0] http://pki-srv01.corp.cvilab.local/ocsp
Certificate Subject: PKI-SRV01.corp.cvilab.local
```

### Step 3 — Test the Revoked Certificate from Lab 01

1. Run:

```powershell
certutil -URL <revoked-certificate.cer>
```

2. Select **OCSP** → click **Retrieve**

Expected result: **Certificate is Revoked**

> **If you get "Good" for the revoked certificate:** The OCSP responder is reading a stale CRL that does not yet include the Lab 01 revocation. Run `certutil -CRL` to force a fresh CRL publication, wait 60 seconds, and retest.

**OCSP response for the REVOKED certificate:**
- [x] Revoked
- [ ] Good (unexpected — investigate using the note above)
- [ ] Unknown
- [ ] Connection timeout / error

```
Status: Failed (Certificate is Revoked)
Type: OCSP
Url: http://pki-srv01.corp.cvilab.local/ocsp
Certificate Subject: PKI-SRV01.corp.cvilab.local
```

### Step 4 — Run Full Certificate Verification on Both Certificates

1. Run certutil -verify on both certificates and record the output:

```powershell
# Valid certificate — should complete without a revocation error
certutil -verify <valid-certificate.cer>

# Revoked certificate — should fail with revocation error
certutil -verify <revoked-certificate.cer>
```

**certutil -verify output for the VALID certificate:**

```
ChainContext.dwInfoStatus = CERT_TRUST_HAS_PREFERRED_ISSUER (0x100)
Leaf certificate revocation check passed
CertUtil: -verify command completed successfully.
```

**certutil -verify output for the REVOKED certificate:**

```
ChainContext.dwErrorStatus = CERT_TRUST_IS_REVOKED (0x4)
Element.dwErrorStatus = CERT_TRUST_IS_REVOKED (0x4)
The certificate is revoked. 0x80092010 (-2146885616 CRYPT_E_REVOKED)
Certificate is REVOKED
Leaf certificate is REVOKED (Reason=1)
CertUtil: -verify command completed successfully.
```

---

## Part E — Inspect the AIA Extension in Context

### Step 1 — Dump a Full Certificate

1. Run a full dump of one of your certificates:

```powershell
certutil -dump <certificate.cer>
```

### Step 2 — Locate the Authority Information Access Section

1. Find the **Authority Information Access** section in the output
2. It contains two distinct entries:

| AIA Entry | Typical URL Format | Purpose |
|---|---|---|
| CA Issuers (caIssuers) | `http://pki-srv01.corp.cvilab.local/CertEnroll/CVI Issuing CA 1.crt` | Tells relying parties where to download the issuing CA's certificate to build the trust chain |
| OCSP (id-ad-ocsp) | `http://pki-srv01.corp.cvilab.local/ocsp` | Tells relying parties where to query real-time revocation status via OCSP |

> **Why both entries matter:** The CA Issuers URL is used for chain building — if a relying party does not have the issuing CA certificate in its trust store, it downloads it from this URL to complete the chain. The OCSP URL is used for revocation checking. A certificate with a missing or unreachable AIA entry will fail validation in environments that require complete chain building or revocation checking.

**CA Issuers URL:**

```
http://pki-srv01.corp.cvilab.local/CertEnroll/PKI-SRV01.corp.cvilab.local_CVI Issuing CA 1.crt
```

**OCSP URL:**

```
http://pki-srv01.corp.cvilab.local/ocsp
```

**Explain in one sentence what each URL is used for by a relying party:**

```
CA Issuers URL: Used by relying parties to download the issuing CA certificate so they can build
and validate the full certificate chain up to a trusted root.

OCSP URL: Used by relying parties to send a real-time query to the Online Responder and receive
a signed response indicating whether the certificate is currently Good, Revoked, or Unknown.
```

---

## Part F — Lab Report

Answer all questions in your own words. Write in complete sentences.

**1. What is the operational difference between a relying party downloading a CRL and sending an OCSP query? From an operational standpoint, what does each approach trade off?**

```
When a relying party downloads a CRL, it retrieves a complete list of all revoked certificates from
the CA's distribution point and caches it locally. This is efficient because one download covers
many certificates, but it introduces a delay — the relying party only learns about new revocations
when it downloads a fresh CRL, which might not happen until the current one expires. CRLs can also
get large in environments with many revocations.

When a relying party sends an OCSP query, it asks the Online Responder about one specific certificate
in real time and gets a signed response back immediately. This is much more current than a cached CRL,
but it adds a network round-trip for every certificate verification and creates a dependency on the
OCSP responder being available. The trade-off is freshness versus efficiency — CRL is bulk and cached,
OCSP is targeted and real-time.
```

**2. The OCSP signing certificate has two properties not found on standard end-entity certificates. Name them, give their OIDs, and explain why each is required.**

```
The first property is the OCSP Signing Extended Key Usage, OID 1.3.6.1.5.5.7.3.9. This EKU is what
tells relying parties that this specific certificate is authorized to sign OCSP responses. Without it,
a relying party would reject the signature on the OCSP response because the signing cert would not be
recognized as having the authority to vouch for certificate status.

The second property is the id-pkix-ocsp-nocheck extension, OID 1.3.6.1.5.5.7.48.1.5. This extension
tells relying parties not to check the revocation status of the OCSP signing certificate itself. Without
it, checking a certificate's revocation status via OCSP would require the relying party to send another
OCSP query to check the signing cert — which would then require another query for that cert, and so on
forever. The nocheck extension breaks this infinite loop by saying "trust this signing cert without
checking its revocation status."
```

**3. Your Online Responder reads the CA's CRL for its revocation data. What happens to the accuracy of OCSP responses if the CRL becomes stale — specifically, what response would a relying party receive for a certificate that was revoked after the last CRL was published?**

```
The Online Responder in this environment uses a CRL-based revocation provider, which means it reads
the CA's published CRL to determine the status of certificates. If a certificate is revoked after the
last CRL was published but before a new CRL is issued, the Online Responder has no way to know about
that revocation yet. A relying party querying for that certificate's status would receive a "Good"
response, even though the certificate has actually been revoked. This is one of the key limitations of
CRL-based OCSP — the freshness of OCSP responses is bounded by the CRL publication interval. In a
production environment, this is addressed by publishing CRLs more frequently or by using Delta CRLs
that the Online Responder can pick up between full CRL cycles.
```

**4. If the OCSP responder on PKI-SRV01 became unreachable, and a relying party was configured for hard-fail revocation checking, what would happen to all certificate verifications in your environment — including valid certificates?**

```
In a hard-fail environment, if the OCSP responder is unreachable, the relying party cannot confirm the
revocation status of any certificate. Rather than assuming the certificate is valid, hard-fail behavior
treats an unknown status as a failure. This means every certificate verification would fail — including
certificates that are completely valid and have never been revoked. Essentially, the entire PKI-dependent
infrastructure would stop working until the OCSP responder came back online. This is why high-availability
OCSP deployments use load-balanced responder arrays, and why some environments fall back to CRL checking
when OCSP is unavailable rather than failing outright.
```

**5. Compare the two test results from Part D: Good for the valid certificate and Revoked for the Lab 01 certificate. What did the OCSP response tell the relying party in real time that a stale CRL could not?**

```
The OCSP response told the relying party the current, authoritative status of each certificate at
the exact moment the query was made. For the valid certificate, it confirmed the certificate was Good
and had not been revoked. For the revoked certificate, it confirmed the certificate was Revoked, with
a signed response from the Online Responder that could be trusted immediately.

A stale CRL could not provide this level of certainty. If a relying party had cached an older CRL
that predated the revocation, it would still show the revoked certificate as valid. The OCSP response
bypassed the caching problem entirely by going directly to the source in real time. This is the core
advantage of OCSP over CRL in time-sensitive scenarios — the response reflects the CA's current
knowledge, not a snapshot from whenever the last CRL was published and cached.
```

---

## Submission Checklist

- [x] Logged in as CORP\pki.admin (not a local account)
- [x] Online Responder role service installed — OCSPSvc status confirmed Running
- [x] Revocation configuration created for CVI Issuing CA 1
- [x] Online Responder snap-in shows green status
- [x] OCSP signing certificate located in the Personal store
- [x] OCSP Signing EKU (1.3.6.1.5.5.7.3.9) confirmed present
- [x] id-pkix-ocsp-nocheck extension (1.3.6.1.5.5.7.48.1.5) confirmed present
- [x] OCSP URL identified from AIA extension
- [x] certutil -URL for valid certificate: Good response documented
- [x] certutil -URL for revoked certificate (from Lab 01): Revoked response documented
- [x] certutil -verify output for both certificates included
- [x] AIA extension entries (CA Issuers URL + OCSP URL) identified and explained
- [x] All five lab report questions answered in complete sentences
- [x] Lab file committed to `labs/week-12/lab-02-ocsp-responder.md`