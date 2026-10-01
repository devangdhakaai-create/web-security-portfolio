# Web App Security Labs — PortSwigger Web Academy

Documenting my hands-on practice solving PortSwigger Web Security Academy labs using Burp Suite (Community Edition) + Turbo Intruder for throttle-free brute forcing.

---

## 🔹 SSRF (Server-Side Request Forgery)

| # | Lab | Status | Writeup |
|---|-----|--------|---------|
| 1 | Basic SSRF against local server | ✅ Solved | [View](./SSRF/01-basic-ssrf-localhost.md) |
| 2 | Basic SSRF against another back-end system | ✅ Solved | [View](./SSRF/02-ssrf-backend-system.md) |
| 3 | SSRF with blacklist-based input filter | ✅ Solved | [View](./SSRF/03-ssrf-blacklist-filter-bypass.md) |
| 4 | SSRF with filter bypass via open redirection vulnerability | ✅ Solved | [View](./SSRF/04-ssrf-openredirect-bypass.md) |
| 5 | SSRF with whitelist-based input filter | ✅ Solved | [View](./SSRF/05-ssrf-whitelist-filter-bypass.md) |

---

## 🔹 SQL Injection

| # | Lab | Status | Writeup |
|---|-----|--------|---------|
| 1 | SQL injection vulnerability in WHERE clause allowing retrieval of hidden data | ✅ Solved | *Pending documentation* |
| 2 | SQL injection vulnerability allowing login bypass | ✅ Solved | *Pending documentation* |
| 3 | SQL injection attack, querying the database type and version on Oracle | ✅ Solved | *Pending documentation* |
| 4 | SQL injection attack, querying the database type and version on MySQL and Microsoft | ✅ Solved | *Pending documentation* |
| 5 | SQL injection attack, listing the database contents on non-Oracle databases | ✅ Solved | *Pending documentation* |
| 6 | SQL injection attack, listing the database contents on Oracle | ✅ Solved | *Pending documentation* |
| 7 | SQL injection UNION attack, determining the number of columns returned by the query | ✅ Solved | *Pending documentation* |
| 8 | SQL injection UNION attack, finding a column containing text | ✅ Solved | *Pending documentation* |
| 9 | SQL injection UNION attack, retrieving data from other tables | ✅ Solved | *Pending documentation* |
| 10 | SQL injection UNION attack, retrieving multiple values in a single column | ✅ Solved | *Pending documentation* |
| 11 | Blind SQL injection with conditional responses | ✅ Solved | [View](./SQL-Injection/11-blind-sqli-conditional-responses.md) |
| 12 | Blind SQL injection with conditional errors | ✅ Solved | [View](./SQL-Injection/12-blind-sqli-conditional-errors.md) |

---

## Tools Used
- Burp Suite Community Edition
- Turbo Intruder (free BApp extension — used as a free alternative to Burp Pro's unthrottled Intruder)

## Notes
- Labs 1–10 (SQLi) were solved before I started proper documentation — writeups for these will be backfilled.
- Labs 11–12 onward are documented in full detail at the time of solving, including payloads, screenshots, and technique breakdowns.

