# Lab: Modifying Serialized Data Types

**Category:** Insecure Deserialization
**Difficulty:** Practitioner
**Status:** ✅ Solved
**Tool used:** Burp Suite Community Edition — Repeater (Inspector panel)

## Objective
Exploit a PHP type-juggling vulnerability in a serialization-based session mechanism to bypass authentication and access the `administrator` account, then delete `carlos`.

## Why This Works
PHP's `==` comparison is loosely typed — when comparing a string against the integer `0`, PHP often evaluates it as `true` due to type coercion rules. The access token is normally a string, checked against a stored value. By changing the token's **data type** from string (`s:`) to integer (`i:0;`) in the serialized object, the loose comparison treats "no valid string" as equal to `0`, bypassing the token check entirely — this is a classic **PHP type juggling** bug, distinct from the simple boolean-flip in the previous lab.

## Steps

1. Logged in as `wiener:peter`, captured `GET /my-account?id=wiener` → sent to Repeater.
2. Decoded the session cookie via Inspector (URL encoding → Base64) to reveal:
   `O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"<random_token>";}`
3. Made **4 precise edits** in the decoded text:
   - Changed username length `6` → `13`
   - Changed value `"wiener"` → `"administrator"`
   - Changed access_token value to `0` and removed the surrounding double-quotes
   - Changed the access_token's type label from `s` (string) → `i` (integer)
4. Final payload: `O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}`
5. Clicked **"Apply changes"**, then **Send**.
6. Response confirmed admin access — "My account" page rendered with `username: administrator` and an **Admin panel** link visible.
7. Navigated to `/admin` → saw user delete controls.
8. Sent `GET /admin/delete?username=carlos` → user deleted.
9. Lab marked **Solved**. ✅

## Key Takeaway
When a serialized session object contains a string field used for sensitive comparisons (tokens, flags), check whether changing its **data type** (not just its value) breaks the authentication logic. PHP's loose `==` comparison makes `0`, `"0"`, `false`, `null`, and even non-numeric strings dangerously interchangeable in security checks — always test type confusion alongside simple value edits when probing serialized data.

## Screenshots
- `01-decoded-cookie-admin-type-juggling-applied.png`
- `02-admin-access-granted-response.png`
- `03-rendered-account-page-administrator.png`
- `04-lab-solved-shop-page.png`