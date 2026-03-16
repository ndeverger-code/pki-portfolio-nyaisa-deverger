# Week 2 Reflection — Cryptography Fundamentals



## 1. What did you learn this week?
This week was all about how we actually build trust online so we aren't just guessing if a site is safe.
* **Confidentiality, Integrity, and Authenticity:** Confidentiality is keeping a secret (locking the door). Integrity is making sure the message wasn't changed (checking the seal). Authenticity is proving the sender is real (checking their ID).
* **Symmetric Encryption:** This is like a lockbox with only one key. You use the same key to lock and unlock it. It's pretty fast and efficient for sending lots of data, but you have to keep that key safe.
* **Hashing and Tampering:** A hash is a one-way "fingerprint" of a file. If someone changes the data, the fingerprint looks totally different, so you know someone messed with it.
* **Digital Signatures:** This combines both. You lock a hash with your private key. It proves the message is original and that it came from you because only you have that specific key.

## 2. What concept was most challenging?
The most challenging part was **differentiating encryption from signing.** At first, I learned and observed signatures are more like a "notary stamp." They don't always hide the words; they just prove they are legit. 

I also had a bit of a challenge **mapping my local files to GitHub.** I had to make sure my work in `C:\Users\nyasi\labs\...` was uploaded to the exact right folder in my repository so my lab submissions stayed organized.
It was a simple tasks, however it happens from time to time!

## 3. Where does this appear in real-world systems?
* **HTTPS Connections:** When I see the padlock icon in my browser, I know my computer is using these tools to talk to the website privately.
* **Software Updates:** When my phone or computer downloads an update, it checks the **Digital Signature** to make sure the code actually came from Apple or Microsoft and isn't a virus.
* **Code Signing:** Developers sign their apps so the operating system knows the code hasn't been tampered with since the developer finished it.

## 4. How would you explain this topic to a non-technical audience?
* **Encryption** is like putting a letter in a safe that only the person with the right key can open.
* **Hashing** is like taking a picture of a seal on a box. If you get the box and the seal doesn't match the picture, you know someone opened it.
* **Digital Signatures** are like a wax seal that only a specific person has the stamp for. It proves the letter is from them and hasn't been opened.

## 5. What questions remain?
I'm still curious about **Key Management.** If a large company has thousands of employees and servers, how are all of the keys tracked? In my 
current role there seems to be a process however a bit faulty. I am curious what an efficient process looks like in that respect. 

---

## Professional Growth Check

- Did you explain concepts in your own words? Yes. I did get some editing asisstance. 
- Did you connect theory to practice? Yes
- Did you reference your lab observations? Yes
- Is your formatting clean and structured? Yes
- Was your commit message meaningful? Yes. It could have been better. I had some challenges with my structure which resulted
- in quite a few commits, however this was mainly due to typos in my path 

---

## Forward Link

Week 2 introduced the mechanisms that enforce digital trust.

Week 3 will show how X.509 certificates combine encryption, hashing, and digital signatures into a structured trust model.

Be prepared to inspect certificates with deeper technical understanding.
