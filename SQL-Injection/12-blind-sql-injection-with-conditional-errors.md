Lab: Blind SQL injection with conditional errors
Category: SQL Injection (Blind)
Platform: PortSwigger Web Academy
Database: Oracle
URL: portswigger.net/web-security/sql-injection/blind/lab-conditional-errors

Goal: Exploit blind SQLi via TrackingId cookie — no direct output, no "Welcome back" signal either; instead, a SQL error (triggered conditionally) is the only oracle. Extract administrator's password and log in.

Vulnerability: TrackingId cookie value is concatenated into a backend Oracle SQL query. The app shows a generic 500 error whenever the query is malformed/throws a runtime error, and 200 OK otherwise. This lets an attacker trigger errors conditionally using CASE WHEN ... THEN TO_CHAR(1/0) ELSE '' END — a true condition causes a divide-by-zero error, a false one doesn't.