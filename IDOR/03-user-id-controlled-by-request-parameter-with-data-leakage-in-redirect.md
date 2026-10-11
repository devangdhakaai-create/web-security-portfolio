# IDOR: User ID Controlled by Request Parameter, with Data Leakage in Redirect

**Platform:** PortSwigger Web Security Academy
**Category:** IDOR — Horizontal Privilege Escalation
**Difficulty:** Apprentice
**Status:** ✅ Solved

## Root Cause (one-liner)
> Server ne redirect (302) bhejne se pehle hi response body mein sensitive data (carlos ka API key) likh diya tha — authorization check "surface pe" theek dikh raha tha (redirect ho raha tha), lekin damage pehle hi ho chuka tha.

**Pattern tag:** `idor-leak-before-redirect`

## Kya Hua, Step by Step

### 1. Apna data dekha — baseline
`wiener` se login karke `/my-account?id=wiener` pe apna data confirm kiya:
API Key: `Cx6OOjwUkBUxJN8D8HFM2BdGDH5fJWN9`

📸 `screenshots/idor-data-leakage-in-redirect/01-own-account-api-key.png`

### 2. Request ko Burp Repeater mein bheja
Burp **Proxy → HTTP History** mein `GET /my-account?id=wiener` request dhundi, **right-click → Send to Repeater**. Yahan se hum request ko manually modify aur resend kar sakte hain, baar-baar — browser ki tarah redirect ko turant follow kiye bina.

📸 `screenshots/idor-data-leakage-in-redirect/02-sent-to-repeater-burp-history.png`

### 3. ID tamper kiya aur raw response dekha
Repeater mein `id=wiener` ko `id=carlos` kiya, **Send** dabaya.

**Response:** `HTTP/2 302 Found`, `Location: /login` — matlab **upar se app secure lag raha hai**, user ko login page pe bhej raha hai jaise koi unauthorized access block ho gaya.

📸 `screenshots/idor-data-leakage-in-redirect/03-id-tampered-302-redirect-raw.png`

### 4. Asli bug — response body check kiya
Yahi par normal testing rukh jaati — "302 redirect mila, access denied, sab theek hai" soch ke. Lekin **response ka pura body bhi dikha** (Repeater browser ki tarah auto-redirect follow nahi karta jab tak tum na bolo). Body render karke dekha toh carlos ka poora data already present tha:

Username: `carlos`
API Key: `o8gSKBxUnwkkSnlVdy8jcFxoyhkicXEr`

Server ne pehle user ka data fetch karke response body mein daal diya, **uske baad** realize kiya "ye galat user hai" aur redirect header add kiya — lekin body already ja chuka tha response ke saath.

📸 `screenshots/idor-data-leakage-in-redirect/04-rendered-body-carlos-api-key-leaked.png`

### 5. Exploit confirm + submit
Leaked API key submit kiya — lab solved.

## Why Ye Vulnerability Hai (Simple Bhasha Mein)
Developer ne authorization check **galat jagah** lagaya — redirect bhej diya (jo normally "access denied" treat hota hai), **lekin data fetch aur response body construction redirect se PEHLE ho chuka tha**. Ye ek **incomplete/broken fix** hai: control flow galat order mein likha gaya.

**Key insight:** Sirf **status code ya final behavior** (redirect = safe) dekh ke assume mat karo ki vulnerability fix hai. Hamesha **poora response (headers + body)** check karo — browser aksar redirect turant follow kar leta hai aur tumhe body dikhne hi nahi deta. Isiliye Burp Repeater zaroori hai — wahan tum manually control karte ho ki redirect follow karna hai ya pehle response dekhna hai.

## Real Bug Bounty Mein Ye Pattern Kaise Dikhta Hai
- APIs jo "access denied" pe redirect karte hain lekin body mein already JSON data bhej dete hain
- Error pages jo debug info/stack trace ke saath sensitive data leak karte hain before showing "unauthorized"
- Mobile apps mein bhi: app redirect follow kar leta hai UI mein, lekin raw response (jo attacker Burp se dekh sakta hai) mein data already leaked hota hai

**Tip:** Jab bhi koi endpoint redirect kare authorization fail hone pe, hamesha raw response body check karo Burp mein — redirect status code pe bharosa mat karo.

## Fix (Report Ke Liye)
Authorization check **sabse pehle** hona chahiye, data fetch/serialize se pehle. Flow ye hona chahiye: (1) verify ownership → (2) agar fail, turant redirect/403 bhejo bina kisi data ko process kiye → (3) tabhi data fetch aur response build karo jab authorization pass ho.