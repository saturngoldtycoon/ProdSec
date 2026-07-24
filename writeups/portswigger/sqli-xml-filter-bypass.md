# SQL Injection with Filter Bypass via XML Encoding

**Source:** PortSwigger Web Security Academy
**Category:** SQL Injection / WAF Bypass
**Difficulty:** Practitioner
**Date:** 2026-07-24

## Objective
Retrieve the administrator's credentials from the `users` table and log in as admin. Data is sent to the app as XML, and a WAF blocks obvious SQLi payloads.

## Recon / Initial Testing
Sent the stock-check request to Burp Repeater. Injecting a normal SQLi payload (e.g. `1 UNION SELECT NULL,NULL--`) into the `storeId` XML field returned a 403 "Attack detected" — WAF is filtering on raw SQL syntax.

## The Vulnerability
The backend concatenates the `storeId` value directly into a SQL query with no parameterization. A WAF sits in front and pattern-matches for obvious SQL syntax, but only inspects the raw (undecoded) request.

## Exploitation Steps
1. Install Hackvertor via the Burp BApp Store.
2. In Repeater, write the injection payload as normal in the XML body.
3. Highlight the payload text, right-click → Extensions → Hackvertor → Encode → `dec_entities`.
4. Send — WAF no longer flags it since the payload is now encoded (e.g. `'` becomes `&#39;`).
5. Backend XML/SQL parser decodes entities back to raw characters before executing the query — payload runs as intended.
6. Determined column count, then used UNION to extract `username`/`password` from `users`.
7. Logged in as administrator.

## Why It Worked
The WAF inspects the request as transmitted (encoded), while the backend decodes it before use. This encode/decode mismatch is a classic WAF bypass pattern — the filter and the interpreter don't see the same string.

## Fix / Mitigation
Parameterized queries (prepared statements) so user input is never concatenated into SQL regardless of encoding. WAF rules alone are not a substitute for fixing the underlying query construction.

## Notes for Next Time
- The lab's hint mentions Hackvertor but doesn't explain how to use it — had to look up the extension's right-click workflow separately.
- General lesson: WAF bypass often comes down to finding an encoding the WAF doesn't decode but the backend does. Worth remembering as a pattern, not just this one lab.
