# Reflection

**1. Which issues were the easiest to fix, and which were the hardest? Why?**

The easiest were the style issues from Flake8 and Pylint: trailing whitespace, the unused variable, `== None`, long lines and missing docstrings. Each one points to an exact line and the fix is obvious. Renaming the functions to snake_case was also easy, but I had to update every call in `main()` as well.

The hardest were the ones that change behaviour. For the SQL injection I had to rewrite the query with placeholders and check that names with quotes (like `O'Brien`) still worked. For `except Exception: pass` I had to decide which exception to catch and what to do with it, instead of just deleting the line. The `rentBike` inconsistent return also needed care because `main()` checked `rentBike(9) is None`, so changing the return value meant changing that check too.

**2. Did the static analysis tools report any false positives? If so, describe one example.**

None were really wrong, but some were lower risk than they looked. Bandit B311 (insecure random) is a warning for any use of `random`. For something like a game it would be a false positive. Here it is a real issue because the number is an unlock PIN, so I fixed it. Also, the tools overlap a lot: the same line-length, whitespace and `== None` problems showed up in both Pylint and Flake8, so the total count looks bigger than the number of real problems.

The tools also missed things. None of them reported that `hasBikes` crashes with a `TypeError` for an unknown station id. I only found that by reading the code around the line Pylint flagged.

**3. How would you integrate static analysis tools into your actual software development workflow?**

- Run Flake8 and Pylint in the editor so problems show up while typing.
- Add a pre-commit hook that runs Flake8, Pylint and Bandit before each commit.
- Run all three in CI (for example GitHub Actions) on every push and pull request, and fail the build on any Bandit Medium/High issue or a Pylint score below a set limit (for example 9.5).
- Keep a shared config file (`.pylintrc`, `setup.cfg`) so everyone uses the same rules.

**4. What tangible improvements did you observe in the code quality, readability, or potential robustness after applying the fixes?**

- Pylint went from 3.89/10 to 10.00/10. Bandit and Flake8 went from 3 and 10 issues to 0.
- Security: station names can no longer inject SQL, and unlock PINs are no longer predictable.
- Robustness: the log list is no longer shared between calls, `rent_bike` always returns `True` or `False`, `has_bikes` returns `False` for unknown stations instead of crashing, and database errors in `reset_station` are reported instead of hidden.
- Readability: consistent snake_case names, docstrings on every function, and shorter lines make the code easier to read and review. The file can also be imported without running the demo.
- The program output is the same as before, so the fixes did not break anything.
