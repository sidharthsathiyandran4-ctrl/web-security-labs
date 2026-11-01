# SQL injection vulnerability allowing login bypass

**Lab:** [PortSwigger Academy](https://portswigger.net/web-security/sql-injection/lab-login-bypass)
**Topic:** SQL injection | **Level:** Apprentice
**Tools:** Burp Suite Community (optional, can be done in the browser)

## Summary

The login form puts the `username` and `password` values directly into a SQL query. This lets an attacker change the query so the password check is skipped and log in as `administrator` without knowing the password.

## How I found it

The login page sends a POST request to `/login` with `username` and `password` parameters. I tried a single quote `'` in the username field to see how the application would react, since an unexpected change in behavior suggests the input is being placed into a query.

## Exploitation steps

1. Opened the **My account** login page.
2. Entered the following username and any value as the password:

```
Username: administrator'--
Password: anything
```

3. The application logged me in as `administrator` and the lab showed as **Solved**.

## Why it works

The backend query likely looks like this:

```sql
SELECT * FROM users WHERE username = 'administrator' AND password = 'anything'
```

After injecting `administrator'--`, the query becomes:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'anything'
```

The `'` closes the username string and `--` turns the rest of the line into a comment. The `AND password = ...` check never runs, so the database returns the administrator row no matter what password was entered.

## Impact

An attacker can log in as any known user, including administrators, without credentials. In a real application this could give full access to user data and admin functions.

## Fix

- Use parameterized queries (prepared statements) so input is always treated as data, not SQL.
- Never build queries by concatenating user input.
- Store passwords as salted hashes and compare them in application code, not in raw SQL.
- Apply least privilege to the database account the app uses.

## What I learned

Comment characters can remove entire conditions from a query. A login check that looks secure in the UI can be bypassed if the backend builds SQL from raw input.
