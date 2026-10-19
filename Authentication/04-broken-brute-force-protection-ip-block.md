## Lab 4: Broken brute-force protection, IP block

**Category:** Authentication (Practitioner)
**Status:** ✅ Solved

**Bug:** Server 3 lagatar fail logins ke baad IP block kar deta tha, lekin
beech me ek **successful login** aane par fail-counter reset ho jaata tha.
Isse attacker apna hi account use karke counter baar-baar reset kar sakta
tha aur IP-block ko bypass karke victim ka password brute-force kar sakta
tha.

**Process jo follow kiya:**
1. `POST /login` request invalid creds se li → Burp Intruder → **Pitchfork attack** select kiya (Sniper nahi)
2. Dono `username` aur `password` params pe `§§` payload position lagaya
3. **Resource Pool → Max concurrent requests = 1** set kiya (order maintain karne ke liye, warna requests random sequence me jaake logic tod deti)
4. **Username list** banayi jisme pattern tha: `wiener, carlos, carlos, wiener, carlos, carlos...` repeat (apna login har 2 carlos attempts ke baad)
5. **Password list** banayi jisme har candidate password **do baar** tha (carlos ke 2 attempts ke liye), aur beech-beech me apna correct password `peter` (wiener ke login ke liye) — dono lists line-by-line **exactly aligned** honi chahiye
6. Attack run kiya → results me **Status 200 hide** kiya filter se
7. Baaki (non-200) results ko **username se sort** kiya → `carlos` ke saamne sirf **ek 302** mila
8. Us row ka **Payload 2 (password)** column check kiya → password mila

**Root cause:** Rate-limiting / brute-force protection sirf failed-attempt
counter pe based tha, aur wo counter kisi bhi successful login (chahe
attacker ke apne account ka ho) se reset ho jaata tha — proper per-account
lockout nahi tha.

**Credentials:** `wiener:peter` (apna), `carlos:matrix` (brute-forced)

**Key takeaway for future labs:** Jab bhi "X failed attempts = block, lekin
counter reset hota hai Y condition se" wala pattern dikhe, **Pitchfork +
Resource Pool (concurrency=1)** combo yaad rakhna — sequence-dependent
logic flaws isi tarike se exploit hote hain.