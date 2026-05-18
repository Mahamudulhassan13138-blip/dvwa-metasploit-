# 🔐 DVWA Vulnerability Writeups

Practice writeups for **Damn Vulnerable Web Application (DVWA)** covering common web vulnerabilities.

> ⚠️ For **educational purposes only**. Use only in an isolated lab environment.

---

## 📁 Modules

| # | Vulnerability | Folder |
|---|--------------|--------|
| 1 | Brute Force | [`brute-force/`](./brute-force/) |
| 2 | Command Injection | [`command-injection/`](./command-injection/) |
| 3 | CSRF | [`csrf/`](./csrf/) |
| 4 | SQL Injection | [`sql-injection/`](./sql-injection/) |
| 5 | XSS Reflected | [`xss-reflected/`](./xss-reflected/) |
| 6 | XSS Stored | [`xss-stored/`](./xss-stored/) |

---

## 🛠 Setup

Run DVWA using Docker:

```bash
docker run -d --name dvwa -p 80:80 ghcr.io/digininja/dvwa:latest
```

Open `http://localhost/setup.php` → Click **Create / Reset Database** → Login: `admin` / `password`

---

## 📖 How to Use

Each folder contains:
- `README.md` — Explanation, payloads, and how the attack works
- `payloads.txt` — Ready-to-use payloads

Set DVWA security level to **Low** first, then try Medium and High.

---

*Made for ethical hacking practice and cybersecurity learning.*
