\# Lab 02: AD CS Console Exploration \& CA Hierarchy Documentation



\*\*Student Name:\*\*  

\*\*Date Completed:\*\*  

\*\*Phase:\*\* 2 | \*\*Week:\*\* 9  

\*\*Submission Path:\*\* `labs/week-09/lab-02-environment-documentation.md`



\---



\## Part A — AD CS Console Exploration (PKI-SRV01)



\### CA Console Nodes — Observations



| Node | Contents / Observations |

|------|------------------------|

| Revoked Certificates | |

| Issued Certificates | |

| Pending Requests | |

| Certificate Templates | |



\### CA Properties — Key Settings



\*\*General Tab\*\*

\- CA Name:

\- Computer Name:



\*\*Extensions Tab — CRL Distribution Points (CDP):\*\*



```

(paste or describe CDP path here)

```



\*\*Extensions Tab — Authority Information Access (AIA):\*\*



```

(paste or describe AIA path here)

```



\*\*Storage Tab\*\*

\- Database Path:

\- Log Path:



\### Certificate Templates Console (certtmpl.msc)



Templates visible in the forest (list what you observed):



```

(list template names here)

```



\---



\## Part B — CA Hierarchy Verification (PKI-SRV01)



\### Command: certutil -store -enterprise Root



```

(paste output here)

```



\*\*What did you see?\*\* (Subject, Issuer, Thumbprint — describe in your own words):



```

(your notes here)

```



\### Command: certutil -store -enterprise CA



```

(paste output here)

```



\*\*What did you see?\*\* (Subject, Issuer, Thumbprint — describe in your own words):



```

(your notes here)

```



\---



\## Part C — Active Directory Structure (DC01)



\### Active Directory Users and Computers (dsa.msc)



\*\*PKI Admins OU — accounts found:\*\*



```

(list accounts here)

```



\*\*pki.admin account — group memberships:\*\*



```

(list groups here)

```



\*\*cert.manager account — group memberships:\*\*



```

(list groups here)

```



\*\*Domain-joined computer accounts found:\*\*



```

(list computer names here)

```



\### Active Directory Sites and Services (dssite.msc)



\*\*Server registered under Default-First-Site-Name:\*\*



```

(server name here)

```



\### Certificate Templates Console (certtmpl.msc on DC01)



Did templates appear here, confirming they are stored in AD?

\- \[ ] Yes

\- \[ ] No — describe what happened:



\---



\## Part D — Environment Summary Write-Up



> Write this section in your own words. Cover all five areas. Aim for clarity over length.



\### 1. Environment Topology



\*(Describe the three VMs, their roles, and IP addresses.)\*



\### 2. CA Hierarchy



\*(Describe the Root CA and Issuing CA. Who issued the Issuing CA's certificate? Why is the Root CA kept offline?)\*



\### 3. Certificate Templates



\*(List the templates currently published to CVI Issuing CA 1. What are one or two of them designed to issue?)\*



\### 4. Active Directory Structure



\*(Describe the PKI Admins OU. Explain the difference between the pki.admin and cert.manager accounts and their roles.)\*



\### 5. One Thing I Found Interesting or Unexpected



\*(Your observation here.)\*



\---



\## Submission Checklist



\- \[ ] Part A: All five console nodes documented

\- \[ ] Part A: CA Properties (CDP, AIA, Storage) recorded

\- \[ ] Part A: Certificate Templates console observed

\- \[ ] Part B: certutil -store -enterprise Root output included

\- \[ ] Part B: certutil -store -enterprise CA output included

\- \[ ] Part C: PKI Admins OU and both accounts documented

\- \[ ] Part C: Sites and Services server noted

\- \[ ] Part C: certtmpl.msc confirmed templates in AD

\- \[ ] Part D: All five summary areas completed in own words

\- \[ ] File committed to portfolio repo at `labs/week-09/lab-02-environment-documentation.md`

