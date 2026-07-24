# SQL Injection in WHERE Clause Allowing Retrieval of Hidden Data

**Source:** PortSwigger Web Security Academy
**Category:** SQL Injection
**Difficulty:** Apprentice
**Date:** 2026-07-24

## Objective
Perform a SQL injection attack that causes the app to display one or more unreleased products.

## Recon / Initial Testing
The product category filter takes a value that gets used in a WHERE clause. Tested by injecting SQL syntax into the category parameter to see if it would alter the query logic.

## The Vulnerability
The app builds a query like `SELECT * FROM products WHERE category = 'X' AND released = 1`, inserting user input directly into the category value with no sanitization or parameterization.

## Exploitation Steps
1. Payload used: `' OR 1=1--`
2. Injected into the category parameter.
3. This closes the string, adds `OR 1=1` (always true), and comments out the rest of the original query with `--`.
4. Since the condition is always true, the `released = 1` check gets bypassed too, returning all products including unreleased ones.

## Why It Worked
`OR 1=1` makes the WHERE clause evaluate true for every row regardless of category, and `--` comments out anything after it (including the `released = 1` check), so the filter is fully neutralized.

## Fix / Mitigation
Use parameterized queries/prepared statements so user input can never alter query structure, regardless of what characters are submitted.

## Notes for Next Time
This is the foundational SQLi payload — good to have memorized cold: `' OR 1=1--`.
