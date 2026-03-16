## 1. Core Concept
Cryptography is the tech that makes digital trust actually work. It uses three main tools: **Encryption** to keep secrets, **Hashing** to make sure nobody changed your files, and **Digital Signatures** to prove who sent a message. It’s the reason things like online banking and secure logins are possible.

## 2. Why It Matters
In the real world, we do everything online, so we need a way to protect private info. Cryptography automatically enforces safety rules that people can't do on their own—like checking that a software update isn't a virus or making sure no one is "listening in" on a private conversation between servers.

## 3. Technical Breakdown

### A. Symmetric Encryption (Confidentiality)
- **Definition:** Using one secret key to lock and unlock data.
- **Components:** Message, scrambled code, and one shared key.
- **Flow:** Message + Key -> Scrambled -> Key + Scrambled -> Message.
- **Trust:** You must keep the key secret; if it's leaked, anyone can read your data.

### B. Hashing (Integrity)
- **Definition:** A one-way math "fingerprint" used to check if data was changed.
- **Components:** Data and a hashing formula (like SHA-256).
- **Flow:** Data -> Math -> Unique Fingerprint. Any tiny change to the data ruins the match.
- **Trust:** It lets you know for sure that nobody messed with your files.

### C. Digital Signatures (Authenticity)
- **Definition:** A digital stamp that proves who sent something and that it’s original.
- **Components:** Private Key (to sign), Public Key (to check), and a hash.
- **Flow:** Lock a hash with a Private Key -> Send it -> Receiver unlocks with a Public Key to verify.
- **Trust:** Proves the sender is real and the data hasn't been tampered with.

## 4. Common Misconceptions
- **Misconception 1:** Encryption and hashing are the same. **Reality:** Encryption is for secrets; hashing is for checking if a file was changed.
- **Misconception 2:** Digital signatures hide your data. **Reality:** They don't hide it; they just "stamp" it so you know it’s legit.
- **Misconception 3:** Asymmetric encryption does all the heavy lifting. **Reality:** It just sets up the connection; symmetric encryption does the actual work of moving data.

## 5. Where This Shows Up
- **Web Security:** The digital signatures on HTTPS certificates.
- **Internal Systems:** Companies using their own "keys" to identify employee laptops.
- **Cloud & DevOps:** Using hashes to verify that code or cloud files haven't been hacked during an update.

## Mental Model
**Identity + Trust + Verification**
- **Encryption** keeps the secret.
- **Hashing** enforces integrity
- **Signatures** prove who it's from.
  
## Part 3 — Observations

### Why the encrypted file is unreadable
The file looks like gibberish because the encryption algorithm scrambled all the original data. Without the right key to put the pieces back in order, it’s just a random mcharacters that doesn't make any sense to a human or a computer.

### What would happen if the wrong password were used
If you use the wrong password, the decryption will fail. The computer will try to unscramble the data using the wrong "math," which usually results in an error message saying the key is incorrect, or it just produces even more scrambled characters.

### What security property symmetric encryption provides
Symmetric encryption provides **Confidentiality**. This means it makes sure that only people with the secret key can read the information, keeping it private from everyone else.

### Why TLS uses symmetric encryption for data transfer
TLS uses symmetric encryption for the actual data transfer because it is way faster and more efficient than asymmetric encryption. Once the two computers agree on a secret key (using the slower asymmetric method), they switch to symmetric encryption so they can send large amounts of data with minimal latency 
