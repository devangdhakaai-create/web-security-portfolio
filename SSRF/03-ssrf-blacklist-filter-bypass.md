Lab: SSRF with blacklist-based input filter
Category: SSRF
Platform: PortSwigger Web Academy
URL: portswigger.net/web-security/ssrf/lab-ssrf-with-blacklist-filter

Goal: Access admin interface at http://localhost/admin and delete user carlos, bypassing two anti-SSRF defenses.

Vulnerability: stockApi parameter fetches server-side with a blacklist filter that blocks known-dangerous hostnames and paths, but the filter decodes input differently than the actual HTTP fetch layer — creating a bypass window.

Steps:

1. Sent stockApi=http://localhost/admin → blocked ("External stock check blocked for security reasons").
2. Tried 127.0.0.1, 127.1, decimal IP (2130706433), octal IP → all blocked (blacklist covers common IP obfuscation too).
3. Found http://127.1/ alone passes (shortened loopback IP not blacklisted).
4. Added /admin path → http://127.1/admin → blocked again (path-based blacklist match on admin).
5. Bypassed path filter using double URL-encoding: encoded a in admin as %2561 (which decodes once to %61, then again to a). Filter only decodes once during its check, so it never sees the literal word admin; the actual fetch layer decodes twice and resolves it correctly.
6. Final payload: stockApi=http://127.1/%2561dmin
7. Got 200 response with rendered admin panel (Users list + Delete links).
8. Sent request → 302 Found, redirected to /admin → user deleted, lab marked Solved.

Payload: stockApi=http://127.1/%2561dmin
Injection point: stockApi body parameter (POST /product/stock)

Takeaway: Blacklist filters check the raw string as written, but the HTTP fetch layer decodes URLs before requesting them — this mismatch is a classic bypass. Double URL-encoding (%2561 → %61 → a) exploits the gap between "what the filter sees" and "what the server actually requests." Real-world WAFs are frequently bypassed the exact same way.
