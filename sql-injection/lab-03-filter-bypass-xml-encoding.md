# SQL injection with filter bypass via XML encoding

**Lab:** [PortSwigger Academy](https://portswigger.net/web-security/sql-injection/lab-filter-bypass-via-xml-encoding)
**Topic:** SQL injection | **Level:** Practitioner
**Tools:** Burp Suite, Hackvertor extension (BApp Store)

## Summary

The stock check feature is vulnerable to SQL injection, and the results are reflected in the response, allowing a UNION attack. A web application firewall (WAF) blocks obvious SQL injection payloads sent directly, but the stock check request is sent as XML, so encoding the payload inside the XML lets it bypass the filter.

## How I found it

Checking stock for a product sends a POST request with an XML body, containing `productId` and `storeId` elements. Sending a normal SQL injection payload directly in `storeId` got blocked by a WAF. Since the body was XML, I installed the **Hackvertor** extension from the BApp Store in Burp, which can wrap a payload in XML entity tags that decode back to the original value server-side.

## Exploitation steps

1. Sent a stock check request to Burp Repeater.
2. Confirmed the number of columns in the query using a UNION attack with incrementing NULLs.
3. Wrote the UNION payload to extract both the username and password in a single column:

```sql
' UNION SELECT username || '~' || password FROM users--
```

4. Selected the payload in Repeater, right-clicked, and used **Hackvertor → Encode → XML entities** to wrap it so the WAF wouldn't flag it as raw SQL.
5. Sent the request. The response showed the combined values, separated by `~`, for every user, including the admin.
6. Split the string on `~` to get the admin's username and password.
7. Logged in as `administrator` with the retrieved password.
8. Lab showed as **Solved**.

## Why it works

The application parses the XML body and extracts the SQL query parameter from it, but the WAF only inspects the raw request text for known SQL injection patterns. Hackvertor wraps the payload as an XML entity, so the literal text sent to the WAF doesn't look like SQL syntax. The XML parser resolves the entity to its real value before that value reaches the database query, so the injection still executes even though the WAF let it through.

The `||` operator concatenates strings. Using `username || '~' || password` combines both columns into the single column the query returns, with `~` as a separator to make the two values easy to split apart when reading the response.

## Impact

An attacker can bypass a WAF meant to block SQL injection, then run arbitrary queries, including reading other tables like `users`. In this lab that led to recovering admin credentials and full account takeover.

## Fix

- Don't rely on a WAF as the only defense. Use parameterized queries (prepared statements) so user input is never treated as part of the SQL syntax, which closes the vulnerability regardless of encoding tricks.
- If a WAF is used, apply it after decoding/parsing steps, not before, so it sees the actual value that reaches the query.
- Disable unnecessary XML processing features unless they're required.

## What I learned

A WAF that filters raw request text can be bypassed by encoding the payload in a format, like an XML entity, that gets decoded later in the processing pipeline before it reaches the vulnerable code. Tools like Hackvertor make this kind of encoding trivial once you understand why it works. String concatenation (`||`) is also a useful trick for pulling two columns worth of data through a query that only returns one column.
