# Week 7 Lesson Notes — Where Certificates Live

## 1. Core Concept

In a real enterprise, TLS certificates don't just sit on a single web server. They live inside
layered infrastructure — CDN edge nodes, load balancers, and application servers can all be
involved in a single connection. The big skill for this week is figuring out *where* TLS
actually terminates. The certificate itself gives you clues: who issued it, what the SANs look
like, and what headers the server sends back can all point to where the encrypted connection ends.

When I analyzed servicenow.com, the cert was issued by Amazon RSA 2048 M04 — an AWS Certificate
Manager intermediate CA. That told me right away that TLS had to end at an AWS layer, because
ACM certs can't be installed anywhere else.

---

## 2. Why It Matters

Knowing where TLS terminates matters for security and operations. If TLS ends at a load balancer,
the traffic between the load balancer and the origin server might be unencrypted — which could
be a risk depending on the network. It also matters for certificate management: if a CDN or
cloud provider is handling your cert automatically, you don't have to worry about renewing it
manually. But if you're managing it yourself, a missed renewal brings the whole site down.



---

## 3. Technical Breakdown

**Where TLS can terminate:**
- **CDN edge** — Cloudflare, Fastly, or CloudFront handles the TLS connection. Headers like
  `CF-Ray` or `X-Cache` show up in server responses.
- **Load balancer** — AWS ALB, F5, or similar. ACM certs are a strong indicator.
- **Application server** — The origin web server handles TLS directly. You'd see Apache or
  Nginx headers with no CDN/proxy headers at all.

**How to identify the CA type from the issuer:**
- Amazon RSA 2048 → AWS ACM (auto-managed, DV only, must be on AWS)
- DigiCert / Entrust / Sectigo → Paid public CA (OV or EV available)
- Let's Encrypt → Free, 90-day, DV only, usually automated
- Internal CA → Private, only trusted inside the organization

**DV vs. OV vs. EV:**
- **DV** — Only proves you own the domain. Subject has no O= field. (e.g., ACM, Let's Encrypt)
- **OV** — Proves domain + company identity. Subject includes O= (Organization name).
- **EV** — Proves domain + legal org identity with strict vetting. Subject has full location info.

**Certificate Transparency (CT) Logs:**
Every public cert has to be logged in CT logs since 2018. crt.sh aggregates them. You can use
CT logs to see how many certs have been issued, what CAs were used, and whether any unexpected
issuers appear. For servicenow.com, the CT logs showed they used Entrust for nearly 10 years
before migrating to Amazon ACM and DigiCert — that kind of history is only visible through CT logs.

---

## 4. Common Misconceptions

- **"The cert I pull is always managed by the org."** — Not if the issuer is ACM or a CDN CA. A third party may be handling it automatically.

- **"Server: Apache means TLS ends at Apache."** — No. ServiceNow showed `Server: Apache` but TLS terminated at an AWS ALB. The origin server header leaks through even when it's behind a layer.

- **"No CDN headers means no CDN."** — Those layers often strip their own headers. Always check the issuer, not just the response headers.

- **"An A- SSL Labs grade means the setup is weak."** — A- is still strong. It usually just means something small is missing, like OCSP Must-Staple or HSTS preload.

---

## 5. Where This Shows Up

- **Enterprise websites** — Most large companies route traffic through a load balancer or CDN
  before it hits an origin server. ServiceNow is a real-world example of this.
- **Healthcare systems** — A hospital patient portal might sit behind an AWS ALB with an ACM cert,
  while internal EHR systems use a private CA completely separate from the public-facing setup.
- **Cloud migrations** — When organizations move to the cloud, their CA relationships change.
  ServiceNow's CT log showed three different CAs active at once during their migration — that's
  normal, and worth knowing before Metro General does the same.
- **Security monitoring** — CT logs are used to catch unauthorized or unexpected certificate
  issuance for a domain, which can be an early sign of a phishing attack or misconfiguration.
- **Internal enterprise systems** — Internal servers, VPNs, and devices use private CAs that
  never appear in CT logs. TLS still terminates somewhere — often an internal load balancer.
- **Cloud environments** — Cloud providers like AWS make cert management easier through tools
  like ACM, but you have to know which layer is handling TLS to manage it correctly.
- **DevOps workflows** — Automated pipelines need valid certs too. If a cert expires mid-deployment,
  the pipeline breaks. Knowing where certs live helps teams plan renewals before that happens.



---

## Mental Model

Think of a TLS certificate like a visitor badge at a secure building. Week 7 adds one question
to the framework from previous weeks: *where does the security check actually happen?*

- **Identity** — The Subject and SANs are the name on the badge. ServiceNow's was DV — just domain ownership, no company name.
- **Trust** — The issuer vouches for the identity. Amazon Root CA 1 is the trusted authority behind the chain.
- **Verification** — This happens where TLS terminates. For ServiceNow, that's the AWS ALB — the actual security checkpoint. Apache is already inside the building.

