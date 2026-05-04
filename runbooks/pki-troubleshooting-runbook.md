# PKI Troubleshooting Runbook

Submit this in your portfolio repository:
`runbooks/pki-troubleshooting-runbook.md`

---

## Title

**TLS Connection Failure — Missing Intermediate CA Certificate (Broken Chain)**

---

## Problem Statement

The server is only sending its own certificate (the leaf) when it connects, but it is missing the middle certificate in the chain (the intermediate CA). Without that middle piece, browsers and clients have no way to trace the certificate back to a trusted authority — so they reject the connection, even though the certificate itself is not expired or invalid.

---

## Environment

| Item | Detail |
|---|---|
| Tool | OpenSSL (`openssl s_client`, `openssl verify`, `openssl x509`) |
| Terminal | Git Bash (recommended) or PowerShell |
| Certificate involved | Server certificate + missing middle (intermediate) certificate |
| Applies to | Any web server, internal portal, or API endpoint returning a TLS error |

---

## Symptoms

- Browser shows a TLS/SSL error or "Your connection is not private" warning
- `openssl s_client` reports: `verify return code: 21 (unable to verify the first certificate)`
- `openssl verify leaf_cert.pem` returns: `error 20 at 0 depth lookup: unable to get local issuer certificate`
- Only **1 certificate** appears in the output — when you would normally expect 2 or 3
- Replacing or renewing the certificate does **not** fix the issue

---

## Diagnostic Steps

**1. Connect to the server and check how many certificates it is sending.**

```
openssl s_client -connect <hostname>:443 -showcerts </dev/null 2>/dev/null
```

Look for lines starting with `depth=` in the output. A healthy chain shows `depth=0`, `depth=1`, and `depth=2`. If you only see `depth=0`, the server is only sending its own certificate — the middle certificate is missing.

**2. Save the server certificate to a file.**

```
openssl s_client -connect <hostname>:443 -showcerts </dev/null 2>/dev/null \
  | openssl x509 -outform PEM > leaf_cert.pem
```

**3. Test whether the server certificate can be validated on its own.**

```
openssl verify leaf_cert.pem
```

If it returns `error 20: unable to get local issuer certificate`, the certificate itself is fine — the problem is that the middle certificate is missing from what the server is sending.

**4. Find where to download the missing middle certificate.**

```
openssl x509 -in leaf_cert.pem -noout -text | grep -A2 "Authority Information Access"
```

Look for a line starting with `CA Issuers - URI:`. Copy that URL — it is where the middle certificate can be downloaded from.

**5. Download the missing middle certificate and convert it to a usable format.**

```
curl -o issuer_cert.der "<URL copied from step 4>"
openssl x509 -inform DER -in issuer_cert.der -out issuer_cert.pem
```

The first command downloads the file. The second converts it from the downloaded format (DER) into the text-based format OpenSSL uses (PEM).

**6. Confirm the two certificates work together correctly.**

```
openssl verify -untrusted issuer_cert.pem leaf_cert.pem
```

Expected output: `leaf_cert.pem: OK` — this means the server certificate and the middle certificate are a valid pair and the full chain traces back to a trusted root.

---

## Resolution

The certificate itself does not need to be replaced. The fix is updating what the server sends during a connection:

1. Combine the server certificate and middle certificate into one file:
   ```
   cat leaf_cert.pem issuer_cert.pem > fullchain.pem
   ```
   This creates a single file (`fullchain.pem`) that contains both certificates back to back.
2. Update the server's TLS configuration to point to `fullchain.pem` instead of the server-certificate-only file.
3. Restart or reload the web server to apply the change.
4. Re-run the check from Step 1 and confirm `depth=0`, `depth=1`, and `depth=2` all appear, with `verify return code: 0 (ok)`.

---

## Prevention Note

After every certificate deployment, run the check from Step 1 and confirm at least two certificates appear in the output before you close out the change. Most certificate issuance emails from the CA include a full chain file — always use that file when configuring the server, not the server certificate alone.

---

**Lab this scenario is drawn from:** [`labs/week-06/submissions/broken-chain/lab-02-broken-chain.md`](../labs/week-06/submissions/broken-chain/lab-02-broken-chain.md)

---

*CVI PKI Career Pathway — Phase 1 Foundations*

---
