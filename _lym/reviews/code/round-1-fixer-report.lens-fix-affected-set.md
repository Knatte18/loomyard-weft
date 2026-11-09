```yaml
lens: fix-affected-set
outcome: reported
```

Findings: none.

- **F2 (blank reader row).** The `noteOrNone(read())` path at `rubric.go:90` is the only site that turns a blank redundancy report into `none`. The new row ("a blank redundancy report renders none") in `TestReadRubric` is the one member the finding named. The bouncer, burler and recipe sites only pass the reader through to `ReadRubric`, so they have no blank-handling of their own to test.
- **F1 (closure branches).** `redundancyReportNote` has two branches the finding named: the read error and the match. The new test `TestRedundancyReportNote_ReadsTheWorktreeReport` covers both with a malformed file and a matching judged report. Its wiring sibling, `TestWire_ReviewKeysReachTheEnv`, still covers the not-found branch. Two branches are untested: the errors from `fabricengine.Open` and `HeadSHA`. The review did not name them, and the render-level `MatchFailed` row already covers how a match error is worded.

Tool calls:
1. `grep` for `noteOrNone` and `RedundancyReport:` across `internal`: my context did not show the other sites that use the redundancy report or `noteOrNone`.
