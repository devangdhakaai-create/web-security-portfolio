Lab: Basic SSRF against another back-end system
Category: SSRF
URL: portswigger.net/web-security/ssrf/lab-basic-ssrf-against-backend-system

Goal: Internal 192.168.0.X range scan karke port 8080 pe admin interface dhundo, fir carlos delete karo.

Vulnerability: stockApi parameter server-side fetch karta hai bina validation ke — attacker internal network scan kar sakta hai (SSRF → internal port/host discovery).

Steps:

1. Intercepted POST /product/stock in Burp, found stockApi param.
2. Used Turbo Intruder (Community edition ka Intruder throttled hai, isliye free unthrottled tool use kiya) to fuzz internal IP range 192.168.0.1–255 on fixed port 8080.
3. Payload: stockApi=http://192.168.0.%s:8080/product/stock/check?productId=1&storeId=2
4. Results table mein anomaly rank + response length se different host identify kiya (baseline se alag response).
5. Confirmed host, then hit /admin on that IP:port → got admin panel with delete links.
6. Final payload (fully URL-encoded):

stockApi=http%3A%2F%2F192.168.0.38%3A8080%2Fadmin%2Fdelete%3Fusername%3Dcarlos

7. Sent → lab solved.

Payload: http://192.168.0.38:8080/admin/delete?username=carlos
Tool used: Burp Suite Community + Turbo Intruder extension (bypass for Intruder's request throttling)
Injection point: stockApi body parameter

Takeaway: SSRF sirf ek fixed internal URL tak limited nahi hota — attacker isse network scanner ki tarah use kar sakta hai poora internal range discover karne ke liye. Encoding (%3A, %2F, %3F, %3D) sahi hona critical hai warna request parse hi nahi hoga.
