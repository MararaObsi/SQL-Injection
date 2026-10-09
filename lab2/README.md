# Lab: SQL injection vulnerability allowing login bypass

- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (SQLi)
- **Difficulty:** Apprentice
- **Target User:** `administrator`

---

## 1. Description & Objective
This lab contains an SQL injection vulnerability in the login function. An attacker can manipulate the database query executed during authentication to bypass the password check and log in as any user.

**Goal:** Exploit the SQL injection vulnerability to log in to the application as the `administrator` user and solve the lab.

---

## 2. Vulnerability Analysis
The backend authentication query is constructed dynamically by concatenating user input directly into the SQL string:

```sql
SELECT * FROM users WHERE username = 'INPUT_USER' AND password = 'INPUT_PASSWORD'