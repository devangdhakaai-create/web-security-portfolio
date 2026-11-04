# Lab: Limit Overrun Race Conditions

**Category:** Race Conditions
**Difficulty:** Apprentice
**Status:** ✅ Solved
**Tool used:** Burp Suite Community Edition (Repeater — Send group in parallel)

## Objective
Exploit a race condition in the discount coupon logic to apply the same 20% coupon multiple times and buy the "Lightweight L33t Leather Jacket" ($1337.00) for less than the available store credit.

## Why This Works
The server checks "is this coupon already applied?" and then applies the discount as two separate steps. If multiple requests hit the server at the same time, they all pass the check before any of them finish updating the database — so the discount gets applied more than once.

## Steps

1. Logged in with `wiener:peter` and added the leather jacket to the cart.
2. Applied the coupon `PROMO20` once via the browser to capture the request in Burp.
3. Sent `POST /cart/coupon` to **Repeater**.
4. Grouped the tab (**Add tab to new group**) and **duplicated it 14 times** (15 total identical requests).
5. Removed the coupon from the cart to reset state.
6. Clicked **"Send group (parallel)"** to fire all 15 requests simultaneously.
7. Checked responses — multiple tabs returned `Coupon applied` instead of `Coupon already applied`, confirming the race condition.
8. Refreshed the cart — price had dropped significantly (discount stacked).
9. Repeated the remove → send group (parallel) cycle a couple more times to stack the discount further.
10. Once the total dropped below available store credit, clicked **Place order**.
11. Lab marked as **Solved**. ✅

## Result
- Store credit: $2.97
- Final order total: **$47.03** (from original $1337.00)
- Coupon stacked ~7–8 times via repeated parallel batches

## Key Takeaway
Burp Suite Community doesn't have the "Trigger race condition" single-packet custom action (Pro-only). Instead, **Repeater's "Send group in parallel"** feature (added in 2023.9+) is a solid free alternative — sending 15–20 duplicate requests at once was enough to consistently win the race window.

## Screenshots
- `01-race-condition-success-coupon-applied.png` — Repeater response showing `Coupon applied` during parallel send.
- `02-lab-solved-order-confirmation.png` — Final order confirmation page showing lab marked Solved with discounted total.