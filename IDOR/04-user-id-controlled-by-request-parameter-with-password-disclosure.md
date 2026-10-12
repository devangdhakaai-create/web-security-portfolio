# IDOR: User ID Controlled by Request Parameter with Password Disclosure

**Platform:** PortSwigger Web Security Academy
**Category:** IDOR — leads to Vertical Privilege Escalation (admin takeover)
**Difficulty:** Apprentice
**Status:** ✅ Solved

## Root Cause (one-liner)
> Password field browser mein masked (dots) dikhta hai, lekin underlying HTML `value=""` attribute mein **plaintext password** server se bhej diya gaya — IDOR se admin ka account page khola toh uska real password bhi mil gaya.

**Pattern tag:** `idor-plaintext-secret-in-html`

## Kya Hua, Step by Step

### 1. Baseline — apna password field dekha
Apne account (`wiener`) mein password field masked (dots) dikh raha tha — normal UI behavior.

📸 `screenshots/idor-password-disclosure/01-own-account-masked-password.png`

### 2. ID ko `administrator` se tamper kiya
URL mein `id=wiener` → `id=administrator` kiya. Page load hua, username `administrator` confirm hua, aur password field mein **lambi masked string** dikhi (normal se zyada characters) — signal mila ki kuch zyada hi data wahan hai.

📸 `screenshots/idor-password-disclosure/02-id-tampered-administrator.png`

### 3. Page source/raw response check kiya (asli find)
Burp Repeater mein response **Raw** dekha. Browser password ko visually mask karta hai, lekin HTML source mein hamesha actual value likhi hoti hai:
```html
<input required type=password name=password value='i73cu74zhxfc1535i02i'/>
```
Admin ka **plaintext password seedha HTML mein** mil gaya — masking sirf cosmetic thi, actual data already client ko bhej diya gaya tha.

📸 `screenshots/idor-password-disclosure/03-password-in-html-value-attribute.png`

### 4. Privilege escalation — login as admin
Log out karke `administrator` / `i73cu74zhxfc1535i02i` se login kiya. Ab asli admin panel access mila (fake/leaked data nahi, real admin session).

### 5. Exploit complete
Admin panel se `carlos` ko delete kiya — lab solved.

📸 `screenshots/idor-password-disclosure/04-solved-carlos-deleted.png`

## Why Ye Vulnerability Hai (Simple Bhasha Mein)
Ye pehle wale IDOR labs se ek level aage hai: **horizontal → vertical escalation**. Pehle sirf doosre normal user ka data leak hota tha (same privilege level). Yahan IDOR ke through admin ka **password hi mil gaya**, jisse seedha **admin account control** mil gaya — horizontal se vertical privilege escalation.

**Key insight:** UI masking (`••••`) **security nahi hai**, sirf visual convenience hai. Form field mein `value=` attribute mein actual data likha hota hai jo **HTML source mein hamesha visible** hota hai, chahe UI kuch bhi dikhaye. Server ko kabhi bhi password jaisa sensitive data **kisi bhi form mein pre-fill nahi karna chahiye**, especially jab IDOR se kisi aur ka account view ho raha ho.

## Real Bug Bounty Mein Ye Pattern Kaise Dikhta Hai
- Pre-filled forms jo "edit profile" ke liye current values dikhate hain — password/API keys/tokens inadvertently include ho sakte hain
- Mobile/web apps jo "change password" page pe current password ko hidden field mein daal dete hain validation ke liye
- **Hamesha IDOR exploit karte waqt pure page source/HTML check karo, na ki sirf visible UI** — masked fields bhi goldmine ho sakte hain

**Impact:** IDOR + plaintext credential disclosure = ek hi bug se **account takeover of any user including admin** — Critical severity in most bounty programs.

## Fix (Report Ke Liye)
1. Primary IDOR fix: ownership check har request pe
2. Secondary (defense-in-depth): password/secrets ko **kabhi bhi** form response mein pre-fill mat karo — "change password" jaisi forms sirf empty field expect karti hain naya password ke liye, current password wahan hona hi nahi chahiye
