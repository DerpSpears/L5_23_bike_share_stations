# Issues Found and Fixed

File: `bike_share_stations.py`

Line numbers refer to the original file. High and medium severity issues are listed first.

| # | Issue | Type | Tool / Code | Line(s) | Description | Fix Approach |
|---|-------|------|-------------|---------|-------------|--------------|
| 1 | SQL injection | Security (Medium) | Bandit B608 | 9 | The INSERT query is built by joining strings, so a station name like `x'); DROP TABLE stations;--` (or just `O'Brien`) breaks or changes the query. | Use a parameterised query with `?` placeholders and pass the values as a tuple. |
| 2 | Mutable default argument | Bug | Pylint W0102 | 8 | `log=[]` is created once and shared across all calls, so every call keeps adding to the same list. | Default to `log=None` and create a new list inside the function. |
| 3 | Broad exception swallowed | Bug / Security (Low) | Pylint W0718, Bandit B110 | 34-35 | `except Exception: pass` hides every error, so a failed reset goes unnoticed. | Catch only `sqlite3.Error` and print an error message. |
| 4 | Insecure random PIN | Security (Low) | Bandit B311 | 22 | `random.randint` is predictable and not safe for unlock PINs. | Use `secrets.randbelow` (`1000 + secrets.randbelow(9000)`), which gives the same 1000-9999 range. |
| 5 | Inconsistent return | Bug | Pylint R1710 | 14-18 | `rentBike` returns `True` on success but falls through to `None` on failure. | Add an explicit `return False`. |
| 6 | Crash on unknown station | Bug | Found during review (Pylint C0121 pointed to the line) | 39-41 | `hasBikes` reads `row[0]` without checking if `row` is `None`, so an unknown id raises `TypeError`. | Check `row is None or row[0] is None` before using the value. |
| 7 | Comparison to None | Style | Flake8 E711, Pylint C0121 | 40 | `row[0] == None` should use identity. | Changed to `is None`. |
| 8 | Unused variable | Style | Flake8 F841, Pylint W0612 | 27 | `summary = len(stations)` is never used. | Removed the line. |
| 9 | Lines too long | Style | Flake8 E501, Pylint C0301 | 5, 9, 15, 17, 26, 33, 39 | Lines exceed 79 characters (PEP 8). | Wrapped the SQL calls across several lines. |
| 10 | Trailing whitespace | Style | Flake8 W291, Pylint C0303 | 27 | Extra spaces at the end of the line. | Removed (the line was deleted with fix 8). |
| 11 | Function names not snake_case | Style | Pylint C0103 | 8, 14, 21, 25, 31, 38 | camelCase names like `addStation` break PEP 8 naming. | Renamed to `add_station`, `rent_bike`, `unlock_pin`, `full_stations`, `reset_station`, `has_bikes`. |
| 12 | Missing docstrings | Style | Pylint C0114, C0116 | 1 and every function | No module or function docstrings. | Added a one-line docstring to the module and each function. |
| 13 | Script runs on import | Good practice | Code review | 52 | `main()` is called at module level, so importing the file runs the demo. | Wrapped it in `if __name__ == "__main__":`. |

## Results

| Tool | Before | After |
|------|--------|-------|
| Pylint | 3.89/10 (22 messages) | 10.00/10 |
| Bandit | 3 issues (1 Medium, 2 Low) | 0 issues |
| Flake8 | 10 issues | 0 issues |

Reports are in `reports/before/` and `reports/after/`.

## Commits

1. Bandit fixes: parameterised SQL, `secrets` for the PIN, specific `sqlite3.Error` handling.
2. Flake8 fixes: wrapped long lines, removed trailing whitespace and the unused variable, `is None`.
3. Pylint fixes: snake_case names, docstrings, mutable default, consistent returns, missing-station check, main guard.
