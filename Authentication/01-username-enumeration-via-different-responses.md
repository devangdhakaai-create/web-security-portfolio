## Lab 1: Username enumeration via different responses

**Category:** Authentication
**Status:** ✅ Solved

**Bug:** Login response different tha valid vs invalid username ke liye —
invalid username pe "Invalid username" aur valid username pe "Incorrect
password" message aata tha. Isse username enumerate ho gaya.

**Request changed:** `POST /login` → `username` param pe Intruder Sniper
attack (candidate usernames list) → Length column se `al` username pakda gaya.
Fir `password` param pe same attack (candidate passwords list) → Status
code 302 se `princess` password pakda gaya.

**Root cause:** Server alag-alag error messages bhej raha tha username aur
password validation ke liye, jisse attacker ko pata chal gaya ki kaunsa
step fail hua.

**Credentials:** `al` / `princess`