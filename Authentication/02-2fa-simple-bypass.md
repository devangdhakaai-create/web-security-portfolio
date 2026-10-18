## Lab 2: 2FA simple bypass

**Category:** Authentication
**Status:** ✅ Solved

**Bug:** Login karte hi (username+password sahi hone par) server session ko
already "authenticated" treat kar raha tha. 2FA sirf ek extra UI step tha,
backend-enforced nahi — isliye `/my-account` URL directly hit karne se
verification code ke bina bhi account access ho gaya.

**Request changed:** Pehle apne (`wiener`) account se 2FA complete karke
URL note kiya: `/my-account?id=wiener`. Fir `carlos:montoya` se login kiya,
2FA verification page pe aaya, lekin code enter karne ke bajaye URL me
`id=wiener` ko manually `id=carlos` kar diya aur directly navigate kiya.
Page load ho gaya — session already valid tha.

**Root cause:** Access control sirf frontend/flow level pe tha, backend
session ne already login ko confirm kar diya tha before 2FA check complete
hone ke. `id` parameter bhi user-controlled tha, jisse IDOR-type access
bhi possible hua.

**Credentials used:** `wiener:peter` (apna), `carlos:montoya` (victim, URL se access)