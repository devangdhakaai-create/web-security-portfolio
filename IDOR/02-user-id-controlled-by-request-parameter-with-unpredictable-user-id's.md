# IDOR: User ID Controlled by Request Parameter, with Unpredictable User IDs

**Platform:** PortSwigger Web Security Academy
**Category:** IDOR — Horizontal Privilege Escalation
**Difficulty:** Apprentice
**Status:** ✅ Solved

## Root Cause (one-liner)
> App ne IDOR "fix" karne ki koshish ki by using random GUIDs instead of simple usernames — lekin GUID khud app ke andar (blog section) publicly leak ho raha tha, aur backend mein ownership check abhi bhi missing tha.

**Pattern tag:** `idor-obscurity-not-security`

## Kya Hua, Step by Step

### 1. Apna account dekha — GUID format confirm kiya
Login karne ke baad `My Account` URL tha:
```
/my-account?id=96cc6b0a-90ae-476b-8aae-1284538feead
```
Yani IDOR same hai pichle lab jaisa, bas ab ID guessable username nahi balki random GUID hai.

📸 `screenshots/idor-unpredictable-user-ids/01-own-account-guid.png`

**Yahan soch:** "Agar ID random/unguessable hai toh IDOR exploit nahi ho sakta" — ye assumption galat hai, kyunki ID **leak** ho sakti hai kahin aur.

### 2. Carlos ka GUID dhunda (information disclosure)
Home page pe blog section mein gaya, carlos ke naam/post pe click kiya. URL mein uska GUID mil gaya:
```
/blogs?userId=8218ec61-b8a8-4b82-bea3-14806ff2d150
```
Blog author links app ke andar hi carlos ka internal user GUID expose kar rahe the — koi guessing nahi lagi, bas app ke andar ek feature se doosre feature mein ID leak ho gayi.

📸 `screenshots/idor-unpredictable-user-ids/02-carlos-guid-found-in-blog.png`

### 3. ID swap karke exploit kiya
`my-account?id=` mein apna GUID carlos ke GUID se replace kiya:
```
/my-account?id=8218ec61-b8a8-4b82-bea3-14806ff2d150
```
Carlos ka data (username + API key: `I9xHmUvdOI88u6jEuXjVQ9Rtwl1Sdrgj`) seedha mil gaya — same missing ownership check jo pichle lab mein tha.

📸 `screenshots/idor-unpredictable-user-ids/03-solved-carlos-api-key.png`

## Why Ye Important Hai — Key Lesson
**Unpredictable/random IDs IDOR ka fix NAHI hain.** Ye sirf "brute-force guessing" ko mushkil banate hain, lekin agar IDs kahin aur app ke features mein (comments, author links, API responses, shared URLs, exported files) **leak/disclose** ho rahi hain, toh attacker unhe collect karke same exploit kar sakta hai.

**Asli fix hamesha same hai:** server-side **ownership verification** — chahe ID guessable ho ya random GUID, farq nahi padta agar backend check hi nahi kar raha "requested ID == session user ka ID?"

## Real Bug Bounty Mein Is Pattern Ka Use
- App ke different features (social feed, comments, shared links, export/PDF metadata, API error messages) mein IDs leak dhundo
- UUID/GUID dekh ke mat ruko — "unpredictable hai toh safe hai" assume mat karo, enumerate karne ke tareeke dhundo (other endpoints, leaked in JS, etc.)
- Burp mein "Search" feature use karke pure site traffic mein kisi known ID (apni khud ki) ko doosri jagah reference hote hue dhundo — reverse approach bhi kaam aata hai

## Fix (Report Ke Liye)
GUID use karna defense-in-depth ke taur pe theek hai (brute-force harder banata hai), lekin **primary control hamesha hona chahiye**: server har request pe check kare `session.user_id == resource.owner_id`, chahe ID format kuch bhi ho.

