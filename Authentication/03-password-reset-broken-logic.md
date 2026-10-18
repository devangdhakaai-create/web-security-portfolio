## Lab 3: Password reset broken logic

**Category:** Authentication
**Status:** ✅ Solved

**Bug:** Password reset form `temp-forgot-password-token` ki value ko
sirf "present hai ya nahi" check karta tha, uski actual validity
(kis user ka token hai) verify nahi karta tha. `username` bhi hidden
form field ke through client-side se bhej raha tha.

**Request changed:** `POST /forgot-password?temp-forgot-password-token=xxx`
request Repeater me li. URL aur body dono jagah se token value empty
ki, aur body me `username=wiener` ko `username=carlos` kar diya.
`new-password-1` & `new-password-2` me naya password daal ke send
kiya → `302 Found` success response mila.

**Root cause:** Server token ko validate nahi karta, sirf naya
password kisi bhi username ke liye set kar deta hai — attacker apna
hi reset flow use karke kisi bhi victim ka password change kar sakta hai.

**Credentials:** `wiener:peter` (apna), `carlos:test` (reset karke banaya)