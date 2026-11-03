```yaml
lens: fix-consistency
outcome: reported
```

Task: check the `_lym/landing/summary.md` edits (fix-consistency lens) against the code and tests they describe.

No findings, with one LOW note.

- **LOW, `summary.md` Tests, Finalize bullet.** The bullet says "reads an unusable token as none", and `finalize_pr_test.go:119` matches it. The Finalize bullet doesn't mention a cancelled context or the empty-task and unusable-origin skips. That is consistent, because those rows are PR-Review only. `finalize_pr_test.go:214` is an empty-task row, but it runs under `on`, so it isn't a rule-read skip. The F1 fix text asked for "the empty-task and unusable-origin skip rows". The PR-Review bullet carries them: `prreview_test.go:195` (empty task) and `:199` (local origin). Nothing needs to change.

The rest agrees with the code:

- **Behavior bullet.** It matches `resolveWithParentRule` (`pullrequestmode.go:108-134`). A cancelled context or transient failure returns an error. A non-token failure with `recordFound` returns a Halt. A token failure, or no record, logs a Warn and counts as no rule.
- **Design decision.** The "lazy and soft" text matches that same code.
- **PR-Review rows.** All named rows exist: `prreview_test.go:168`, `:174`, `:183`, `:189` and `:466`.
- **Finalize rows.** All named rows exist: `finalize_pr_test.go:108`, `:113` and `:119`.

Calls made:
1. `grep` of the test names across `internal/landingshed/*_test.go`, to confirm each row named in the summary exists. My context didn't hold the tests.
2. `sed` of `prreview_test.go` lines 193-215, plus a grep of the Finalize rows, to confirm the unusable-origin skip row and any Finalize skip rows. My context didn't hold them.
