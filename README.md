# Web App Security Labs — PortSwigger Web Academy

Documenting my hands-on practice solving PortSwigger Web Security Academy labs using Burp Suite (Community Edition) + Turbo Intruder for throttle-free brute forcing.

## Progress

| Topic | Solved | Status |
|-------|--------|--------|
| IDOR / Broken Access Control | 5 | ✅ Documented |
| SSRF | 5 | ✅ Documented |
| SQL Injection | 12 | ✅ Solved (1–10 writeups pending) |
| Authentication | 4 | 🔄 In progress |
| Race Conditions | 0 | ⏳ Next |
| Insecure Deserialization | 0 | ⏳ Planned |
| XXE | 0 | ⏳ Planned |
| API Security | 0 | ⏳ Planned |
| Cloud Misconfigurations | 0 | ⏳ Planned |
| Business Logic Flaws | 0 | ⏳ Planned |

---

## 🔹 IDOR / Broken Access Control

| # | Lab | Status | Writeup |
|---|-----|--------|---------|
| 1 | User ID controlled by request parameter | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/tree/main/IDOR) |
| 2 | User ID controlled by request parameter, with unpredictable user IDs | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/tree/main/IDOR) |
| 3 | User ID controlled by request parameter with data leakage in redirect | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/tree/main/IDOR) |
| 4 | User ID controlled by request parameter with password disclosure | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/tree/main/IDOR) |
| 5 | Insecure direct object references | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/tree/main/IDOR) |

---

## 🔹 SSRF (Server-Side Request Forgery)

| # | Lab | Status | Writeup |
|---|-----|--------|---------|
| 1 | Basic SSRF against local server | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/blob/main/SSRF/01-basic-ssrf-localhost.md) |
| 2 | Basic SSRF against another back-end system | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/blob/main/SSRF/02-ssrf-backend-system.md) |
| 3 | SSRF with blacklist-based input filter | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/blob/main/SSRF/03-ssrf-blacklist-filter-bypass.md) |
| 4 | SSRF with filter bypass via open redirection vulnerability | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/blob/main/SSRF/04-ssrf-openredirect-bypass.md) |
| 5 | SSRF with whitelist-based input filter | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/blob/main/SSRF/05-ssrf-whitelist-filter-bypass.md) |

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
| 11 | Blind SQL injection with conditional responses | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/blob/main/SQL-Injection/11-blind-sqli-conditional-responses.md) |
| 12 | Blind SQL injection with conditional errors | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/blob/main/SQL-Injection/12-blind-sqli-conditional-errors.md) |

---

## 🔹 Authentication

| # | Lab | Status | Writeup |
|---|-----|--------|---------|
| 1 | Username enumeration via different responses | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/tree/main/Authentication) |
| 2 | Username enumeration via subtly different responses | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/tree/main/Authentication) |
| 3 | Username enumeration via response timing | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/tree/main/Authentication) |
| 4 | Broken brute-force protection, IP block | ✅ Solved | [View](https://github.com/devangdhakaai-create/web-security-portfolio/tree/main/Authentication) |

*Remaining auth labs postponed (Burp Community limitations); will resume later.*

---

## Tools Used

- Burp Suite Community Edition
- Turbo Intruder (free BApp extension, used as an alternative to Burp Pro's unthrottled Intruder)

## Notes

- SQLi labs 1–10 were solved before I started proper documentation; writeups will be backfilled.
- Labs 11 onward are documented in full detail at the time of solving, including payloads, screenshots, and technique breakdowns.