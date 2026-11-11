```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for the findings whose premise is a set (F4, F3, F1) cover every member.

**F4, report-format literals (producer and parser).** The grep over non-test Go in `internal` finds no other literal of `its output:`, `... N more lines cut` or `, failed outside any test`. The remaining hits are the `gateslot` constants, the `gatecli` uses of them, and prose in doc comments. Covered.

**F3, `TierRunFailures` callers.** The only non-test caller is `internal/landingshed/verifygate.go:304`. Its doc comment and the `landingshed/doc.go:252` line both state the new rule.

**F1, `Finalize` constructors.**
- Many test fixtures build `&Finalize{...}` literally. They are `finalize_test.go`, `finalize_verify_test.go:65`, `finalize_merge_test.go:106` and others.
- `finalize_verify_test.go:65` and `finalize_merge_test.go:106` set a `gate` but leave `onFlakyPass` nil.
- Both match the old state. Neither expects a flaky rerun, so the fix does not require them to change.

**F5, F6, F7.** Each is a single named site, so there is no wider set to cover.

No findings.

Calls made:
- A `grep` over `internal` for the report-format literals, as the F4 and F3 search.
- A `grep` for `TierRunFailures(` callers and `&Finalize{` literals, as the F3 and F1 search. My context had no cross-package view of either.
