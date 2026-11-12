# Lab: Modifying Serialized Objects

**Category:** Insecure Deserialization
**Difficulty:** Apprentice
**Status:** ✅ Solved
**Tool used:** Burp Suite Community Edition — Repeater (Inspector panel)

## Objective
Exploit a PHP serialization-based session mechanism to escalate privileges from a normal user to admin, then delete the user `carlos`.

## Why This Works
The session cookie is a Base64 + URL-encoded serialized PHP object containing user data, including an `admin` boolean flag (`b:0` = false). Since the server trusts this cookie's contents without re-validating privileges server-side, directly flipping `admin` to `b:1` grants admin access.

## Steps

1. Logged in as `wiener:peter`, captured `GET /my-account?id=wiener` (contains the session cookie) → sent to Repeater.
2. In Repeater's **Inspector panel**, selected the cookie value and used the "+" decode button to reveal layers: URL encoding → Base64 → raw PHP serialized object:
   `O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}`
3. Edited `b:0` → `b:1` directly in the decoded view, clicked **"Apply changes"** (auto re-encodes the cookie).
4. Clicked **Send** — response now included an `/admin` panel link, confirming privilege escalation.
5. Changed the request path to `GET /admin` (keeping the modified cookie) → Send → got the admin panel HTML with delete links.
6. Changed path to `GET /admin/delete?username=carlos` → Send → got `302 Found`, redirecting to `/admin`.
7. Lab marked **Solved**. ✅

## Challenges Faced (remember for next time)
- **Decoding layers:** Burp's Inspector doesn't auto-detect the full decode chain — had to manually select "Base64" from the dropdown after the URL-encoding layer to reveal the actual PHP object.
- **Forgot to click Send:** "Apply changes" only updates the request in the editor — it does **not** send it. A separate click on **Send** is required afterward.
- **Testing in the wrong session:** Visiting `/admin` in the normal browser tab failed ("Admin interface only available if logged in as an administrator") because the browser holds its own unmodified session. All privilege-escalated requests must be sent from within Repeater using the edited cookie — not replicated in the browser.

## Key Takeaway
Never trust client-side serialized data for authorization. Applications must re-validate privileges server-side rather than trusting flags embedded in a client-controlled (even if encoded) session object. This is the most common, easy-to-spot real-world deserialization bug — worth checking cookies/session tokens for PHP serialized data (`O:`, `s:`, `b:`, `i:` patterns) on every engagement.

## Screenshots
- `01-decoded-cookie-admin-b1-modified.png`
- `02-admin-delete-carlos-302-success.png`
- `03-browser-admin-access-denied.png`
- `04-cookie-decoded-admin-b0-original.png`
- `05-my-account-request-with-cookie.png`