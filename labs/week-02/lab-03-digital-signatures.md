# Lab — Digital Signatures (Authenticity)

## Overview
In this lab, I learned how digital signatures work and how they prove both authenticity and integrity. I created a private and public key pair, signed a file, and then verified the signature. I also tested what happens when the file is changed after signing.

---

## Environment
- Operating System: Windows  
- Terminal Used: Git Bash (MINGW64)  
- OpenSSL Version: OpenSSL (used via terminal)

---

## Steps Performed
1. I created a folder for signatures and made a file called `artifact.txt` with a message inside it.  
2. I generated an RSA private key and then created a public key from it. After that, I signed the file using my private key.  
3. I verified the signature using the public key, then modified the file and tried to verify it again to see what would happen.

---

## Results

- I successfully generated a private key and public key:
  - `private_key.pem`
  - `public_key.pem`

- I signed the file and created a signature file:
  - `artifact.sig`

- When I verified the original file, the output showed:

- Verified OK


- After I changed (tampered with) the file and tried to verify again, I got an error:

Verification failure

along with OpenSSL error messages.

- This proved that even a small change to the file breaks the signature.

---

## Key Findings
- A digital signature proves both who created the file and that it hasn’t been changed.
- The private key is used to sign, and the public key is used to verify.
- If the file is modified after signing, the verification fails.

---

## Explanation
These results matter because digital signatures are used in real-world security systems like HTTPS, software downloads, and email security. They make sure that data comes from a trusted source and hasn’t been altered. This connects to PKI because certificates and trust systems rely on signatures to verify identity and protect data.

---

## Challenges / Troubleshooting
- I initially had trouble with long file paths, so I had to carefully check each command.
- Understanding the difference between the private key and public key took some time, but practicing the steps made it clearer.
- The verification failure after tampering looked like an error at first, but I realized it was expected and actually proved the system was working.

---

## Artifacts
- `artifact.txt` (original file)
- `public_key.pem` (public key)
- `artifact.sig` (digital signature file)
- Lab write-up file (`digital-signatures-lab.md`)
- Screenshot of terminal output stored in `assets/screenshots/week-02

---

If you include screenshots, store them in `assets/screenshots/` at the root of your repo and reference them here.

https://github.com/ndeverger-code/pki-portfolio-nyaisa-deverger/blob/main/assets/screenshots/week-02/Digital%20Certificates%20Week%202.png



**Option B — GitHub Web (Easiest)**

Open your `.md` file on GitHub, click the pencil icon to edit, then **drag and drop your image directly into the text editor**. GitHub will upload it automatically and insert the correct link for you.

<img width="3433" height="624" alt="image" src="https://github.com/user-attachments/assets/bf026f17-b5ac-437d-b30e-7e255b1b616d" />


---

## Key Findings
- A digital signature proves both who created the file and that it hasn’t been changed.
- The private key is used to sign, and the public key is used to verify.
- If the file is modified after signing, the verification fails.

---

## Explanation
These results matter because digital signatures are used in real-world security systems. They make sure that data comes from a trusted source and hasn’t been altered. This connects to PKI because certificates and trust systems rely on signatures to verify identity and protect data.

---

## Challenges / Troubleshooting
- I initially had trouble with long file paths, so I had to carefully check each command.
- Understanding the difference between the private key and public key took some time, but practicing the steps made it clearer.
- The verification failure after tampering looked like an error at first, but I realized it was expected and actually proved the system was working.

---

## Artifacts
- `artifact.txt` (original file)
- `public_key.pem` (public key)
- `artifact.sig` (digital signature file)
- Lab write-up file (`digital-signatures-lab.md`)
- Screenshot of terminal output stored in `assets/screenshots/week-02

---

*CVI PKI Career Pathway — Foundations Phase*
