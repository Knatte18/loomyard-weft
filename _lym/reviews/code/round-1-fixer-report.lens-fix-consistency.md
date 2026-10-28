```yaml
lens: fix-consistency
outcome: reported
```

Task: check whether commit a9768a3c (F1 and F2 fix) still agrees with its sibling code, docs and tests.

No findings.

- Code: both `-v` and `--verbose` entries are `false`, agreeing with each other and with `--all` and `-a`.
- Comments: the list-mode comments are generic and need no edit.
- Docs: `doc.go:303` names `-v` beside `-a`, `-r` and `--format`.
- Tests: the new rows match the review fix text.
- Sibling references: a grep of `internal`, `contracts`, `docs`, `PATTERN.md` and `pattern` found no other place treating `-v` as list mode.

Tool calls used (2 of 6):
1. `git show --stat HEAD` plus a grep for other `-v` or `--verbose` references beside `git branch`. My context did not show the commit file list or any file outside `internal/burlerengine`.
