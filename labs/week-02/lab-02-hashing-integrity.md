
# Lab — Hashing Integrity

## Overview
Briefly describe the purpose of this lab in your own words.
What PKI concept or system behavior were you investigating?

In this lab, I learned how hashing works and how it helps detect if data has been changed. I used SHA-256 to create a “fingerprint” of a file, then I changed the file to see how the hash would change. This helped me understand data integrity, which is an important concept in security and PKI.
---

## Environment
Document the environment used to complete the lab.

## Environment
- Operating System: Windows
- Terminal Used: Git Bash (MINGW64)
- OpenSSL Version

---

## Steps Performed
1. I created a folder for my hashing lab and made a file called `message.txt` with a short message inside it.  
2. I generated a SHA-256 hash of the file using OpenSSL and saved the output to `message.sha256.txt`.  
3. I edited the file by adding the word "tampered", then generated a new hash and saved it as `message_tampered.sha256.txt` to compare the results.

---

## Results
Include the important outputs or findings from the lab.

- After running `ls -R labs`, I saw my folder structure was created correctly:
  The hashing command created a SHA-256 output file:
   message.sha256.txt
- After modifying the file, I generated a second hash:
  message_tampered.sha256.txt
 - Final validation using `ls` showed:
message.sha256.txt message.txt message_tampered.sha256.txt


If you include screenshots, store them in `assets/screenshots/` at the root of your repo and reference them here.


**Option B — GitHub Web (Easiest)**

Open your `.md` file on GitHub, click the pencil icon to edit, then **drag and drop your image directly into the text editor**. GitHub will upload it automatically and insert the correct link for you.



<img width="1342" height="559" alt="image" src="https://github.com/user-attachments/assets/ca8e2189-0c11-45ec-9d76-cc8ca173949a" />

---

## Key Findings
- A hash acts like a unique fingerprint for a file.
- Even a very small change (like adding one word) creates a completely different hash.
- SHA-256 always produces a fixed-length output, no matter the size of the input file.

---

## Explanation
These results matter because hashing is used to make sure data has not been changed. In real-world security, this helps verify files, passwords, and messages. If the hash changes, it means the data was modified. This is important in PKI and cybersecurity because it helps detect tampering and ensures trust in data.

---

## Challenges / Troubleshooting
- I accidentally entered a wrong directory path at first, which caused an error (`Is a directory`). I fixed it by correcting the path.
- Remembering the full file paths was a little confusing, but I solved it by carefully checking my folder structure.
- Making sure I used `>>` instead of `>` when modifying the file was important so I didn’t overwrite the original content.


---

## Artifacts

- `message.txt` (original file created for hashing)
- `message.sha256.txt` (SHA-256 hash of the original file)
- `message_tampered.sha256.txt` (SHA-256 hash after the file was modified)
- Lab write-up file (`hashing-lab.md`)
- Screenshot of terminal output stored in `assets/screenshots/week-02

---

*CVI PKI Career Pathway — Foundations Phase*
