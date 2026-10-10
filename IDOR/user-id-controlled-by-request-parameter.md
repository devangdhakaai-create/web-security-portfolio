# IDOR: User ID Controlled by Request Parameter

**Platform:** PortSwigger Web Security Academy
**Category:** IDOR (Insecure Direct Object Reference) — Horizontal Privilege Escalation
**Difficulty:** Apprentice
**Status:** ✅ Solved

## Root Cause (one-liner)
> Server `id` parameter pe blindly trust kar raha tha — ye check nahi kiya ki logged-in user (session) aur requested `id` same hai ya nahi.

**Pattern tag:** `idor-missing-ownership-check`

## Kya Hua, Step by Step

### 1. Normal flow observe kiya
`wiener` account se login karne ke baad, `My Account` page ka URL tha:
```
/my-account?id=wiener
```
Page pe apna API key dikha: `osIS26hgjtlFqPSUPuLS0XG1sJAf1wJn`

📸 `screenshots/idor-user-id-request-parameter/01-own-account-api-key.png`

**Yahan suspicious kya laga?** Username seedha URL parameter mein visible tha. Matlab server frontend se "kiska data chahiye" ye info le raha tha — backend khud decide nahi kar raha apne session se.

### 2. ID parameter tamper kiya
`id=wiener` ko manually `id=carlos` kiya address bar mein, URL submit kiya. Koi login/password nahi diya carlos ka — sirf parameter change kiya.

**Result:** Server ne bina kisi authorization check ke seedha `carlos` ka data return kar diya — uska username aur API key dono.

📸 `screenshots/idor-user-id-request-parameter/02-id-swapped-carlos-api-key-leaked.png`

### 3. Exploit confirm + submit
Leaked API key (`HfaWSwL3NNuJ6kUSsEPSrUAgqwmu42ai`) ko lab ke "Submit solution" box mein daala — lab solved mark ho gaya.

📸 `screenshots/idor-user-id-request-parameter/03-solved.png`

## Why Ye Vulnerability Hai (Simple Bhasha Mein)
Server ne 2 cheezon ko mix up kar diya:
- **Authentication** = "tum kaun ho" (login check ho gaya — session valid tha)
- **Authorization** = "tumhe YE specific data dekhne ka haq hai ya nahi" (ye check **missing** tha)

Login hona kaafi nahi hota — server ko **har resource request pe** verify karna chahiye ki requested object (yahan `id=carlos`) **requesting user (wiener) ka hi hai ya nahi**. Yahi check missing tha, isliye IDOR.

## Real Bug Bounty Mein Ye Pattern Kaise Dikhta Hai
- `GET /api/user/12345/profile` → `12345` ko `12346` karke try karo
- `GET /invoice?id=1001` → sequential IDs guess karke doosre users ka invoice data
- Mobile app APIs mein bhi same pattern common hai (often less tested by devs)

**Impact:** Sensitive data leakage (yahan API key — jo account takeover tak le ja sakta hai agar key se authentication bypass ho sake), typically **High** severity rated in bounty programs.

## Fix (Report Mein Likhne Ke Liye)
Server-side pe **har request** par verify karo: `session.user_id == requested_id`. Agar match nahi karta → 403 Forbidden return karo. Kabhi bhi client-supplied ID ko directly database query mein use mat karo bina ownership verify kiye.
