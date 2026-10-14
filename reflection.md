# Lab 5: Reflection

**File analysed:** `recruitment_tracker.py`

| Tool | Before | After |
|------|--------|-------|
| Pylint | 4.47/10 (21 messages) | 10.00/10 |
| Flake8 | 8 messages | 0 |
| Bandit | 3 issues (2 Medium, 1 Low) | 0 |

## 1. Which issues were the easiest to fix? Which were the hardest?

**Easiest:** the style and cleanup items. Removing the unused variable `total_seen` also removed the trailing whitespace on the same line. Renaming `addApplicant`, `updateStatus`, `exportReport` and `countByStatus` to snake_case, adding docstrings, wrapping long lines and adding a blank line before `exportReport` were all mechanical, and each one was fixed in seconds.

**Hardest:** the `eval(salary_rule)` call. Bandit suggests `ast.literal_eval`, but that would not work here: `literal_eval` rejects an arithmetic expression like `"500000 + 150000"`, which is exactly what `main()` passes. The real fix was to change the design: `shortlist` now takes a numeric `max_salary`, and the caller does the arithmetic. The SQL injection fix was also more than a one-line change, because the INSERT had to be rewritten with `?` placeholders and the salary needed validating. Choosing what to do with the silent `except Exception: pass` also took a decision: I catch only `sqlite3.Error` and log it, so failures are visible but the program does not crash.

## 2. Did the tools report any false positives?

Nothing in this file was a strict false positive; every finding pointed at something real. Some findings were noise or overlap rather than useful signal:

- The same problem was reported by several tools. `eval` appeared in both Bandit (B307) and Pylint (W0123), and long lines appeared in both Flake8 (E501) and Pylint (C0301). The two tools also disagree on the limit (79 vs 100 characters), so the counts are inflated unless they are configured to match.
- Bandit's B608 fires on any SQL built from strings. It is correct here because user input goes into the query, but it would also fire on a harmless query built from constants, which would then need a `# nosec` comment after review.
- Some messages carry little weight, such as the missing-docstring warning on a tiny `main()` function. They matter for maintainability, but they are not bugs.

## 3. How would you integrate static analysis into a real development workflow?

- **In the editor:** run Flake8 and Pylint (for example through VS Code extensions) so problems show up as I type.
- **Pre-commit hooks:** run Flake8, Pylint and Bandit before each commit so bad code never enters the repository.
- **CI pipeline:** run all three tools on every pull request (for example in GitHub Actions) and block the merge on Bandit findings of Medium severity or higher or on a Pylint score below an agreed threshold.
- **Shared configuration:** keep one config (`.pylintrc`, `setup.cfg` or `pyproject.toml`) with a single agreed line length so the tools do not contradict each other.
- **Handling exceptions:** allow a suppression comment such as `# nosec` or `# pylint: disable=...` only with a written reason, reviewed like any other change.

## 4. What tangible improvements did you see after the fixes?

- **Security:** the SQL injection is closed. On the original code, a name like `Eve', 'analyst', 1, 'hired') --` made the program store a row with salary 1 and status `hired`. With placeholders, the same text is stored as an ordinary name. Arbitrary code execution through `eval` is also gone.
- **Correctness:** `shortlist` always returns a list instead of sometimes returning `None`. Database errors are logged instead of silently hidden. The CSV export now closes the file safely and handles commas in names correctly.
- **Maintainability:** consistent naming, docstrings, no dead code and `main()` guarded by `if __name__ == "__main__":` make the file easier to read, test and import.
- **Metrics:** Pylint went from 4.47 to 10.00, and Flake8 and Bandit went to zero findings, while the program's output stayed the same (`['Nina']` and `2 in interview`).

Static analysis is a good first filter, but it does not replace testing or code review. None of the tools flagged the missing salary validation, and only Pylint noticed the inconsistent return.
