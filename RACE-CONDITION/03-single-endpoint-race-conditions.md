# Lab: Single-Endpoint Race Conditions

**Category:** Race Conditions
**Difficulty:** Practitioner
**Status:** ✅ Solved
**Tool used:** Burp Suite Community Edition — Repeater (Send group in parallel)

## Objective
Claim `carlos@ginandjuice.shop` (an address with pending admin invite) by exploiting a race condition in the email-change confirmation flow, then use admin access to delete `carlos`.

## Why This Works
The server kicks off an email-send task using the pending email address, but reads that address from the database *after* the task starts. If a second request overwrites the pending email during this window, the confirmation link gets sent to the wrong inbox — but tied to the newly-overwritten (target) address.

## Steps

1. Captured `POST /my-account/change-email` and sent to Repeater.
2. Created 2 requests in a group:
   - Request A: `email=wiener@exploit-<ID>.exploit-server.net` (own inbox)
   - Request B: `email=carlos@ginandjuice.shop` (target)
3. Sent group in parallel — initial attempts failed (confirmation kept going to own email).
4. **Scaled up to 10 requests** (5x each) in the same group and resent in parallel — 80/20 fix: more duplicate requests = higher chance of hitting the race window on Community edition.
5. Checked exploit server's email client — got a confirmation mail **sent to own inbox but referencing `carlos@ginandjuice.shop`** in the body. ✅ Race condition triggered.
6. Clicked the confirmation link → account email updated to `carlos@ginandjuice.shop`.
7. "Admin panel" link appeared on account page.
8. Opened admin panel → deleted user `carlos`.
9. Lab marked **Solved**. ✅

## Key Takeaway
When a 2-request race fails consistently on Community edition, **duplicate both requests 5x+ and resend in parallel** — this was the deciding factor here, same pattern as the earlier coupon-stacking lab.

## Screenshots
- `01-email-client-race-condition-triggered.png`
- `02-repeater-request-A-email-change-wiener.png`
- `03-repeater-request-B-email-change-carlos.png`
- `04-lab-solved-admin-user-deleted.png`