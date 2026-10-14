# Lab 5: Issues found in recruitment_tracker.py

Line numbers refer to the ORIGINAL file.

| Issue | Type | Line(s) | Description | Fix Approach |
|-------|------|---------|-------------|--------------|
| SQL injection (B608) | Security (Medium) | 8 | `addApplicant` builds the INSERT with an f-string, so a name like `Eve', 'analyst', 1, 'hired') --` rewrites the row's role, salary and status. | Use a parameterised query with `?` placeholders and pass values as a tuple. |
| Use of `eval` (B307, W0123) | Security (Medium) | 14 | `eval(salary_rule)` executes any string passed in as Python code (arbitrary code execution). | Remove `eval`; `shortlist` takes a numeric `max_salary` and the caller passes `500000 + 150000`. |
| Silent `except Exception: pass` (B110, W0718) | Bug / Security (Low) | 26-29 | Any database error in `updateStatus` is swallowed, so failed updates look like successes. | Catch `sqlite3.Error` only and log it with `logging.error`. |
| Inconsistent return (R1710) | Bug | 12-20 | `shortlist` returns a list when there are matches but implicitly `None` when there are none, so callers must special-case it. | Always return a list (empty if no matches); use a list comprehension. |
| File not closed safely, no encoding (R1732, W1514) | Bug / Quality | 32-35 | `open()` without `with` leaks the handle if a write fails; encoding is platform-dependent; `",".join(row)` breaks on commas or None values. | Use `with open(..., encoding="utf-8", newline="")` and `csv.writer`. |
| Unused variable `total_seen` (F841, W0612) | Quality | 15 | Assigned but never used. | Delete it. |
| Non-snake_case function names (C0103) | Style | 7, 24, 31, 38 | `addApplicant`, `updateStatus`, `exportReport`, `countByStatus` break PEP 8 naming. | Rename to `add_applicant`, `update_status`, `export_report`, `count_by_status` and update callers. |
| Missing docstrings (C0114, C0116) | Style | 1, 7, 12, 24, 31, 38, 42 | Module and every function lack docstrings. | Add a one-line docstring to each. |
| Lines too long (E501, C0301) | Style | 4, 8, 13, 26, 39 | Lines exceed 79 (Flake8) / 100 (Pylint) characters. | Wrap strings and arguments across lines. |
| Trailing whitespace (W291, C0303) | Style | 15 | Spaces after `len(rows)`. | Removed with the unused variable. |
| Wrong blank lines (E302) | Style | 31 | One blank line before `exportReport` instead of two. | Add a second blank line. |
| No input validation / unguarded `main()` | Robustness (not tool-reported) | 7, end of file | Salary is never validated, and `main()` runs on import. | Validate salary is a non-negative int; use `if __name__ == "__main__":`. |
