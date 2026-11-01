# security__testing




# SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

**Lab:** [PortSwigger Academy](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)
**Topic:** SQL injection | **Level:** Apprentice

## Summary
The product category filter builds its SQL query by inserting the
`category` parameter directly into the query string. This lets an
attacker change the query and see products that are meant to be hidden.

## How I found it
Selecting a category sent a request like `GET /filter?category=Gifts`.
In Burp (Proxy > HTTP history) I sent this request to Repeater and
tested the `category` parameter.

## Exploitation steps
1. Sent the category request to Repeater.
2. Changed the parameter to: `category=Gifts'--`
3. The response now included products that were not previously shown.
   (Also tested `' OR 1=1--` to return everything.)

## Why it works
The backend query is:
`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`
The `'` closes the string early and `--` comments out the rest of the
line, which removes the `AND released = 1` check. The database no longer
filters out unreleased products.

## Impact
An attacker can read data the application intended to hide. In a real
app, the same flaw could let them read other tables, such as user
credentials.

## Fix
Use parameterized queries (prepared statements) so user input is always
treated as data and never as SQL. Validate input as a secondary control.

## What I learned
Small changes to a single parameter can change the logic of a query.
Burp Repeater makes it easy to test variations quickly.
