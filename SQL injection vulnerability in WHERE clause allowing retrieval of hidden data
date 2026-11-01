# SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

**Lab:** [PortSwigger Academy](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)
**Topic:** SQL injection | **Level:** Apprentice
**Tools:** Burp Suite Community

## Summary

The product category filter inserts the `category` parameter directly into a SQL query. This lets an attacker change the query and view products that are meant to be hidden (unreleased).

## How I found it

Selecting a category sent a request like this:

```http
GET /filter?category=Gifts
```

I sent it to Burp Repeater (Proxy > HTTP history) and tested the `category` parameter with different inputs.

## Exploitation steps

1. Sent the category request to Repeater.
2. Changed the parameter to:

```
category=Gifts'--
```

3. The response now included products that were not shown before. I also tested the following, which returns everything:

```
category=' OR 1=1--
```

4. The lab showed as **Solved**.

## Why it works

The backend query is:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

After injecting `Gifts'--`, it becomes:

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

The `'` closes the string early and `--` comments out the rest of the line. That removes the `AND released = 1` check, so the database no longer filters out unreleased products.

## Impact

An attacker can read data the application intended to hide. In a real application, the same flaw could let them read other tables, such as user credentials.

## Fix

- Use parameterized queries (prepared statements) so input is always treated as data, never as SQL.
- Validate input as a secondary control.

## What I learned

Small changes to a single parameter can change the logic of a query. Burp Repeater makes it easy to test variations quickly.
