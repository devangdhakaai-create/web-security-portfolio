# Unprotected Admin Functionality

**Platform:** PortSwigger Web Security Academy
**Category:** Broken Access Control
**Difficulty:** Apprentice
**Status:** ✅ Solved

## Root Cause (one-liner)
> App assumed the admin panel URL being "secret" was enough protection — no actual authentication/authorization check existed on the `/administrator-panel` endpoint itself.

**Pattern tag:** `missing-auth-check-via-security-through-obscurity`

## Objective
Delete the user `carlos` via an admin panel that should not have been reachable by a low-privilege/unauthenticated user.

## Steps Taken

### 1. Information disclosure via robots.txt
Checked `/robots.txt` on the domain root — this file is public by design (tells search crawlers what *not* to index), so developers sometimes leak sensitive paths into it thinking "robots won't crawl it = it's hidden." It discloses a `Disallow:` entry pointing to the admin panel path.

*(First attempt appended `/robots.txt` to the current page path by mistake — `robots.txt` must always be fetched from the domain root, not a sub-path. Corrected by hitting the bare domain root.)*

📸 `screenshots/unprotected-admin-functionality/01-wrong-robots-path-attempt.png`

### 2. Direct navigation to disclosed admin path
Replaced the path with the disclosed admin panel path (`/administrator-panel`). No login prompt, no role check — the page loaded directly, listing all users (`wiener`, `carlos`) with a `Delete` action next to each.

📸 `screenshots/unprotected-admin-functionality/02-admin-panel-accessed-not-solved.png`

### 3. Exploitation
Clicked `Delete` next to `carlos`. Server executed the deletion with zero authorization check on who's allowed to hit that endpoint.

📸 `screenshots/unprotected-admin-functionality/03-carlos-deleted-solved.png`

## Why This Matters (Bug Bounty Context)
This is the most basic form of **broken access control**: sensitive functionality reachable by *anyone* who knows (or guesses/discovers) the URL. In real programs, hunters find these via:
- `robots.txt`, `sitemap.xml`, JS source files (leaked API paths)
- Common admin path wordlists (`/admin`, `/administrator-panel`, `/dashboard`) via directory brute-force (ffuf/gobuster)
- Archived URLs (Wayback Machine)

**Impact if found in the wild:** Full account takeover / data manipulation / user deletion by an unauthenticated attacker — typically rated **Critical/High** depending on what the admin panel exposes.

## Remediation (for report write-up)
- Enforce server-side authentication + role-based authorization check on every admin route (never trust that the URL is "unguessable")
- Never rely on hiding paths via robots.txt as a security control — robots.txt is publicly fetchable and explicitly *not* a security boundary