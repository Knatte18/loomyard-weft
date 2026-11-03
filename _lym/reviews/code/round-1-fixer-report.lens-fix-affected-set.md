```yaml
lens: fix-affected-set
outcome: reported
```

**Task:** check that the F1–F4 fixes cover every member of each finding's set (callers of `Board.Rename`, literal record-file-name sites, help texts for `front`/`on_land`).

Findings: no findings.

- **F1, callers of `Board.Rename`:** the only non-test caller is `internal/boardcli/cli.go:241`, and the fix handles `*DecisionRecordError` there. The other hits are tests in `decision_test.go`, `rename_test.go`, `taskguard_test.go`, `handles_test.go` and `slugrule_test.go`. None of them asserts an empty result on the record-error path.
- **F3, literal record file names:** no non-test code forms `decision-%s.md` or a `decision-<slug>.md` string any more, apart from the `DecisionFileName("<slug>")` call in `cli.go`. The remaining hits are doc comments (`task.go:63`, `doc.go:292` and `doc.go:353`, `config.go:200`, `replay.go:70`, `decision.go:3`). The `decision-record.md` hits are a different file.
- **F2, help text for `front`, `on_land` and `discussed`:** only `boardcli/cli.go:521` carries the help line. `boardcli/doc.go:29` and `boardengine/doc.go:38` are package docs and are unaffected.

Tool calls made:
1. `grep` for `.Rename(` callers across `internal` and `cmd`, to enumerate F1's caller set.
2. `grep` for literal `decision-…md` names in non-test code, to enumerate F3's sites.
3. `grep` for `front`/`on_land` descriptions in non-test code, to enumerate F2's help sites.
