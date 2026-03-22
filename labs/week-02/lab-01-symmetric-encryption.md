# Lab — Symetric Encryption

## Overview
Briefly describe the purpose of this lab in your own words.
What PKI concept or system behavior were you investigating?

The purpose of this lab is to use one secret key to lock and unlock data.

In this lab, the primary focus was on confidentiality(CIA) through the use of Symmetric Encryption.

## Environment
Document the environment used to complete the lab.

- Operating System: Windows
- Terminal Used: Git Bash
- OpenSSL Version: OpenSSL 3.5.5 27 

---

## Steps Performed
Summarize the key steps you performed to complete the lab.

Do **not copy the lab instructions**.
Describe what you actually did.

1. I created a text file with a message
I used a command to create a file called plaintext.txt and added the text "Week 2 Symmetric Encryption Lab - CVI" inside it.

2. I verified the file was created
I ran the ls command to check the folder contents and confirmed that plaintext.txt was there.

3. I encrypted the file using OpenSSL
I used an openssl command with AES-256 encryption to convert the readable file into an encrypted file (plaintext.txt.enc). I entered a password to secure it.


---

## Results
Include the important outputs or findings from the lab.

Examples may include:

- Command outputs
- Certificate fields or values
- Verification results
- Screenshots (if applicable)

If you include screenshots, store them in `assets/screenshots/` at the root of your repo and reference them here.

**How to embed an image:**

**Option A — Terminal / Local Editor**

Save your screenshot to `assets/screenshots/` in your repo, then reference it using a relative path from your submission file:

```markdown
![Description of your screenshot](../../../assets/screenshots/your-filename.png)
```

> The `../../../` moves up three levels: `submissions/` → `week-03/` → `labs/` → repo root, then into `assets/screenshots/`.

**Option B — GitHub Web (Easiest)**

Open your `.md` file on GitHub, click the pencil icon to edit, then **drag and drop your image directly into the text editor**. GitHub will upload it automatically and insert the correct link for you.

Example of what an embedded image looks like:

```markdown
![Certificate output showing SAN field](../../../assets/screenshots/san-field.png)
```
<img width="586" height="396" alt="image" src="https://github.com/user-attachments/assets/1822fc78-30e1-416b-8e10-a58d97e6fffd" />

---

## Key Findings
Document the most important observations from the lab.

- The encryption command successfully converted readable text into unreadable data using AES-256-CBC. The output included the header Salted__, confirming that a salt was applied during encryption.
- Entering and verifying the password was required to complete the encryption process, showing that symmetric encryption depends on a shared secret.
- The encrypted file (plaintext.txt.enc) could not be read in plain text form, confirming that the encryption worked as intended.

---

## Explanation
Explain **why the results matter**

## Explanation
These results matter because they demonstrate how symmetric encryption protects data. The use of a password means only someone with the correct key can decrypt the file.

* **The Salted__ header** shows that a salt was added, which prevents attackers from easily guessing the password.
* **Real-world security connection:** Encryption is widely used to protect sensitive data such as files, communications, and stored credentials.


---

## Challenges / Troubleshooting
Document any issues encountered during the lab and how you resolved them.

One challenge was ensuring the correct file path was used in the command. A mistake in the path was making my 
commit fail. I created the folder and had typos in them. I corrected the typos and this resolved the issue. 

---

## Artifacts
List the files generated or submitted during this lab.

* **plaintext.txt**: The original file containing the readable message.
* **plaintext.txt.enc**: The encrypted output file generated using AES-256-CBC.
* **Terminal Screenshot**: Documenting the OpenSSL command execution and successful encryption.


---

*CVI PKI Career Pathway — Foundations Phase*
