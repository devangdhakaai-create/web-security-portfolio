# IDOR: Insecure Direct Object References (Classic)

**Platform:** PortSwigger Web Security Academy
**Category:** IDOR — the textbook/original version
**Difficulty:** Apprentice
**Status:** ✅ Solved

## Root Cause (one-liner)
> Chat transcripts server ke file system pe **sequential/incrementing numbered filenames** (`1.txt`, `2.txt`, `3.txt`...) ke saath store hote the, **bina kisi ownership/auth check ke** — koi bhi number guess karke kisi aur ka transcript padh sakta tha.

**Pattern tag:** `idor-sequential-filename-no-auth`

## Kya Hua, Step by Step

### 1. Apna transcript generate kiya
"Live chat" tab mein message bheja, "View transcript" dabaya. Server ne automatically ek transcript file download ki.

📸 `screenshots/idor-classic-insecure-direct-object-references/01-live-chat-transcript-generated.png`

### 2. Pattern notice kiya — incrementing filenames
Browser ke **Downloads panel** mein dekha: pehle se `2.txt`, `3.txt` jaisi files already downloaded thi (previous actions se). Clear pattern dikha — **sirf ek number increment ho raha hai**, koi random/unguessable token nahi.

📸 `screenshots/idor-classic-insecure-direct-object-references/02-downloads-panel-incrementing-filenames.png`

**Yahan IDOR ka signal kya tha?** Jab bhi URL ya filename mein ek **simple sequential number** dikhe (1, 2, 3...), turant socho: "agar main ye number badal doon, kya mujhe kisi aur ka data mil sakta hai?"

### 3. Number 1 try kiya — purana transcript mila
Direct URL access kiya:
```
/download-transcript/1.txt
```
Ye sabse purana/pehla transcript nikla — **social engineering conversation** jisme koi user "Hal Pline" (automated support bot) se apna password confirm karwa raha tha. Bot ne confirm kar diya:
```
Hal Pline: Takes one to know one
You: Ok so my password is oitno902fi2bfr9mrtgb. Is that right?
Hal Pline: Yes it is!
```
Password mil gaya: **`oitno902fi2bfr9mrtgb`**

📸 `screenshots/idor-classic-insecure-direct-object-references/03-transcript-1-txt-password-leaked.png`

### 4. Login kiya aur solve kiya
`carlos` / `oitno902fi2bfr9mrtgb` se login kiya — lab turant solved mark ho gaya, My Account page confirm karta hai `username: carlos`.

📸 `screenshots/idor-classic-insecure-direct-object-references/04-solved-logged-in-as-carlos.png`

## Why Ye Vulnerability Hai (Simple Bhasha Mein)
Ye IDOR ka **purest/original form** hai jisse ye naam (Insecure **Direct Object** Reference) aaya:
- **Direct object** = file (`1.txt`) seedha file system se serve ho rahi hai, koi abstraction layer nahi
- **Insecure reference** = filename khud predictable hai aur **koi authorization check nahi** ki requester ka us specific transcript se koi connection hai ya nahi

Is lab mein ek **interesting twist** bhi hai: actual exploit sirf technical nahi tha — purane transcript mein **social engineering** dikhi (ek user ne khud apna password bot ko bata diya "confirm" karwane ke liye). Yani IDOR ne hume ek **doosra vulnerability class bhi expose kar diya** — real world mein aise chains common hote hain.

## Real Bug Bounty Mein Ye Pattern Kaise Dikhta Hai
- File download endpoints jisme filename/ID URL mein visible ho (`/files/invoice_1042.pdf`, `/export/report-7.csv`)
- Support chat transcripts, uploaded documents, generated reports — sab jagah sequential ya predictable naming check karo
- **Hamesha try karo:** current number se **upar aur neeche dono directions** mein — purane records (1, 2, 3) aksar zyada sensitive hote hain (testing data, early users, admin setup logs)

**Impact:** Yahan direct account takeover hua via leaked credentials — depending on data sensitivity, is class ki vulnerabilities **Critical to High** rated hoti hain bounty programs mein.

## Fix (Report Ke Liye)
1. File access endpoints pe bhi **ownership verification** mandatory karo — session user ko verify karo ki wo transcript uska khud ka hai
2. Predictable sequential IDs ki jagah **unguessable random tokens** use karo (lekin yaad rakho — jaisa lab #2 mein dekha, ye akela fix nahi hai, ownership check hamesha zaroori hai)
3. File system se direct serving avoid karo bina access-control layer ke beech mein
