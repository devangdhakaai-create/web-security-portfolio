🧪 Lab 14 — Blind SQL Injection with Time Delays

Category: SQL Injection (Blind) · Difficulty: Practitioner · Status: ✅ Solved

🎯 Objective

Trigger a 10-second conditional delay via the TrackingId cookie to confirm blind SQLi — no visible output, no errors, just timing.

🔍 Vulnerability

The TrackingId cookie value is concatenated directly into a backend PostgreSQL query. The app gives zero feedback (no errors, no content change), so the only way to confirm injection is by making the database stall on purpose.

💉 Payload

``` TrackingId=x'||pg_sleep(10)-- ```
* `x'` → closes the original string
* `||` → PostgreSQL concatenation operator
* `pg_sleep(10)` → the time-based oracle
* `--` → comments out the rest of the original query

⚙️ Steps
1. Intercept the request in Burp Repeater.
2. Replace the entire `TrackingId` cookie value with the payload above.
3. Hit Send and watch the response time.

✅ Result

Response time jumped from ~400ms baseline to 20,490ms. Lab marked solved.

⚠️ Gotcha I Hit

First attempt failed — forgot the trailing `--`, so the original query's closing quote never got commented out → SQL parse error → `pg_sleep()` never ran, fast response. Always check the Raw tab in Repeater, not Pretty, to catch truncation/encoding issues.

🛠️ Fix

Use parameterized queries everywhere, including data from cookies — not just form fields. Never trust input based on its source.

📸 Screenshots

Folder: screenshots/lab-14/

* sqli-14-vulnerable-request.png — Repeater request with payload in TrackingId cookie
* sqli-14-delayed-response.png — Response time showing 20,490ms delay
* sqli-14-lab-solved.png — Browser confirmation "Congratulations, you solved the lab!"