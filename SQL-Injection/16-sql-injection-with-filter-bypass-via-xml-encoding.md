🧪 Lab 16 — SQL Injection with Filter Bypass via XML Encoding

Category: SQL Injection (WAF Bypass) · Difficulty: Practitioner · Status: ✅ Solved

🎯 Objective

Bypass a WAF protecting the stock-check feature, then extract all usernames and passwords from the users table to log in as administrator.

🔍 Vulnerability

The POST /product/stock endpoint sends productId and storeId as XML. The storeId value gets evaluated server-side in a SQL query, and results come straight back in the response — classic UNION-based injection. The twist: a WAF sits in front and blocks any request that looks like an obvious SQLi attempt.

⚙️ Steps

1. Confirm the injection point Change <storeId>2</storeId> to: ```xml <storeId>1+1</storeId> ``` Response returns stock for store "2" — confirms the value is evaluated server-side.

2. Trigger the WAF (on purpose) ```xml <storeId>1 UNION SELECT NULL</storeId> ``` → 403 Forbidden, body: "Attack detected". Confirms the WAF is pattern-matching plain SQLi syntax.

3. Bypass with Hackvertor Install the Hackvertor extension (BApp Store). Select the payload text, right-click → Extensions → Hackvertor → Encode → hex_entities. This wraps the payload in a Hackvertor tag: ```xml <storeId><@hex_entities>1 UNION SELECT NULL</@hex_entities></storeId> ``` On send, Hackvertor converts it to XML hex entities on the wire — invisible to the WAF's pattern matching, but decoded normally by the server's XML parser.

4. Confirm bypass Send it → 200 OK, response: "0 units" (expected — NULL isn't a real stock value, but no WAF block this time).

5. Extract credentials Replace the payload inside the Hackvertor tag with: ``` 1 UNION SELECT username || '~' || password FROM users ``` → Response returns all usernames and passwords, ~-separated, one per line.

6. Log in Use the administrator row's credentials on the login page.

✅ Result

``` administrator~jo8l6lj5z73ma3lz4tri carlos~vhu4gg6e78fjokcvtvzt wiener~ov1u93hefrw7im8sv9cw ``` Logged in as administrator. Lab marked Solved.

⚠️ Gotcha I Hit

First extraction attempt returned "0 units" even with the UNION working — missing single quotes around the ~ separator (|| ~ || instead of || '~' ||). Without quotes, ~ gets parsed as an operator/identifier, the query fails syntactically, and the app silently shows a generic "0 units" rather than an error. Always quote string literals explicitly, even for a simple separator character.

🛠️ Fix
Use parameterized queries — this closes the injection entirely, making the WAF unnecessary as a primary defense.
A WAF is a detective/preventive layer, not a fix — any signature-based filter can be bypassed via encoding tricks (XML entities, URL encoding, case variation, comments, etc.). Don't rely on it alone.
Validate and strictly type input (e.g. enforce storeId as a plain integer) before it ever reaches the query layer.