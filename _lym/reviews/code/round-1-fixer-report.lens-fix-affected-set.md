```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix for F1 and F2 covers every member of each finding's set.

No findings.

- F1 (set: checkouts that apply `isJudgementPath`): the search finds exactly two call sites, `judgesnapshot.go:207` (code) and `judgesnapshot.go:220` (records). The records side already had the `ArchivedJudgementIsNoChange` row. The new `ArchivedJudgementOnTheCodeCheckoutIsNoChange` row covers the code side, so both are covered.
- F2 (set: negative rows that guard the `archivedFrom == ...` comparison): the added `_lym/drift/other-20261106T083132Z.yaml` is the only shape that passes the stamp, extension and directory checks and fails only on the base comparison. It is in both the `others` list and the expected changes, in sorted position (`old/` < `other-`), so the set is covered.

Tool calls:
1. `grep -rn isJudgementPath`: confirmed the call-site set; my context had only the one file, not the whole repo.
