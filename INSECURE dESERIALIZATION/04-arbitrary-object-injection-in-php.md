# Lab: Arbitrary Object Injection in PHP

**Category:** Insecure Deserialization
**Difficulty:** Practitioner
**Status:** ✅ Solved
**Tool used:** Burp Suite Community Edition — Repeater + Decoder

## Objective
Forge a completely new serialized PHP object (not just edit an existing session) and inject it into the cookie to trigger a dangerous method (`unlink()`) on the server, deleting `/home/carlos/morale.txt`.

## The Core Concept (why this matters beyond the lab)

In PHP, certain class methods are called **"magic methods"** — they run automatically at specific points in an object's lifecycle, without you explicitly calling them:
- `__construct()` — runs when object is created
- `__destruct()` — runs when object is destroyed (e.g. script ends, variable unset)
- `__wakeup()` — runs right after `unserialize()`
- `__toString()`, `__call()`, etc.

**The vulnerability:** when a web app calls `unserialize()` on data it doesn't fully control (like a session cookie), PHP will happily instantiate *any class already loaded in memory* if the serialized string names it — and if that class has `__destruct()` or `__wakeup()`, that method fires automatically, with attacker-controlled property values.

In this lab, `CustomTemplate::__destruct()` calls `unlink($this->lock_file_path)`. Since `lock_file_path` is just a property inside the object, and the object itself is built from a string *we* control, we can:
1. Find any class in the app's source that has a dangerous magic method.
2. Hand-craft a serialized object of that class, setting the dangerous property to whatever we want.
3. Smuggle it into the session cookie.
4. The moment the app unserializes our cookie, the destructor fires, and our attacker-controlled file path gets deleted.

This is **not a class we're "logging in as"** — we never authenticate as `CustomTemplate`. We're just forging raw bytes that PHP will blindly turn into a live object.

## Steps

1. Logged in as `wiener:peter`. In the site map, found a reference to `/libs/CustomTemplate.php`.
2. Sent `GET /libs/CustomTemplate.php` to Repeater, appended a tilde (`~`) to the filename:
   `GET /libs/CustomTemplate.php~ HTTP/2`
   → This is a classic backup-file disclosure trick (many editors/servers leave a `~` backup copy readable as plaintext even when the `.php` extension itself would get executed, not shown).
3. Response leaked the **full PHP source**. Found the dangerous sink:
   ```php
   class CustomTemplate {
       private $template_file_path;
       private $lock_file_path;
       public function __construct($template_file_path) {
           $this->template_file_path = $template_file_path;
           $this->lock_file_path = $template_file_path . ".lock";
       }
       public function __destruct() {
           // ... calls unlink($this->lock_file_path) when object is destroyed
       }
   }
   ```
4. In **Burp Decoder**, manually typed the forged serialized object:
   `O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}`
   - `O:14:"CustomTemplate"` → object, class name is 14 chars
   - `:1:` → object has 1 property
   - `s:14:"lock_file_path"` → property name, a string, 14 chars
   - `s:23:"/home/carlos/morale.txt"` → property value, a string, 23 chars
5. Encoded it: **Encode as → Base64**, then **Encode as → URL** (two layers, matching how the real cookie is encoded).
6. Captured any request with a session cookie (`GET /my-account`), sent to Repeater.
7. **Replaced the entire cookie value** with the forged, double-encoded object from Decoder.
8. Clicked **Send**.

## Why the 500 Internal Server Error Happened (and why that's OK)

Got `HTTP/2 500 Internal Server Error` on sending the forged cookie. This is **expected, not a failure**:

- The app's normal session is a `User` object, with specific properties like `username`, `access_token` it expects to read right after unserializing.
- We replaced it with a `CustomTemplate` object instead — a completely different shape.
- The destructor still fires and deletes the file successfully **during the unserialize step**, but *afterward*, the rest of the request-handling code tries to treat the object as a `User` (e.g. read `$session->username`), which doesn't exist on `CustomTemplate` → PHP throws an error → 500 response.

**The deletion already happened before the crash.** The 500 is just the app falling over *after* the exploit already worked, because our forged object doesn't match what the rest of the code expects going forward. Always check the lab's "Solved" status directly instead of assuming a 500 means failure.

## Challenges Faced (remember for next time)
- **Typos in handcrafted payloads are fatal.** `CoustomTemplate` (extra letters) and `moral.txt` (missing a letter) initially broke things — PHP's length-prefixed serialization format is unforgiving: one wrong character count and the whole string fails to parse.
- **Double encoding is mandatory**, not optional — Base64 alone leaves `=` padding and other characters that break when sent raw in a `Cookie:` header. Always Base64 → then URL-encode, matching the original cookie's own encoding scheme.
- **A crash after the exploit fires is not the same as the exploit failing.** Confirm success via the actual lab state / file existence / app behavior, not just the HTTP status code.

## How to Spot & Exploit This in Real Bug Bounty Targets
1. **Identify deserialization entry points:** session cookies, API tokens, cached data, file upload metadata — anywhere a server reconstructs objects from client-supplied data (look for `unserialize()` in PHP, `ObjectInputStream.readObject()` in Java, `pickle.loads()` in Python, `Marshal.load` in Ruby).
2. **Get source code access** if at all possible — via leaked backup files (`.php~`, `.bak`, `.old`), exposed `.git` directories, error messages leaking stack traces, or a public GitHub repo. Without seeing the classes, you're blind to what magic methods exist.
3. **Hunt for dangerous magic methods** in any class the app loads: file operations (`unlink`, `fopen`, `include`), SQL queries built from properties, `eval()`, command execution, or object instantiation based on a property value (this is how gadget chains work in later labs).
4. **Forge the object by hand** using the language's native serialization syntax — you don't need the exploited class to be "yours" or something you're authorized to use; if PHP's autoloader can find the class definition, your forged bytes are enough.
5. **This generalizes beyond deleting files** — the same technique applies to triggering SSRF, arbitrary writes, or even remote code execution, depending on what the dangerous magic method actually does with the attacker-controlled property.

## Key Takeaway
Insecure deserialization's real danger isn't "the cookie got tampered with" — it's that **unserializing attacker-controlled data can instantiate arbitrary classes and auto-trigger their lifecycle methods**, regardless of whether those classes were ever meant to be user-facing. Source code access (even partial, via leaked files) is often the deciding factor between "theoretically vulnerable" and "actually exploitable" — always prioritize finding leaked source in real engagements before giving up on a deserialization lead.

## Screenshots
- `01-source-code-leak-tilde-trick.png`
- `02-decoder-forged-object-base64.png`
- `03-cookie-replaced-500-error-response.png`
- `04-lab-solved-confirmation.png`