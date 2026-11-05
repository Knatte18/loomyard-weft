```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F2: every site that tests `Recoveries >= maxRecoveries` now also requires `!rendersNewAmendment`. They are `beginbatch.go:421`, `recoverbatch.go:509` (this one also adds `!waitsOnParent`, which the review's own fix key leaves out), `recordbatch.go:227` and `runlevel.go:1204`. `exhaustedRecoveryWayForward` is called only from the three way-forward helpers (which now take the narrowed flag) and the `recoverbatch.go:510` refusal.
- F1: `create-not-done` and `rename-not-done` are keyed only in `cardNotDoneFor` outside `internal/planglyph`, and the one `createNotDone` flag feeds the one `createTargetWayForward` consumer. The `rendersNewAmendment` and `exhausted` sets show no member the fix missed.
- Left unchecked: the test rows and the `doc.go` and `recordbatch.go` doc-comment sentences. Neither finding's premise is a set over them.

Calls made:
1. `grep` for the `Recoveries >= maxRecoveries` sites outside tests. My context held only `recordbatch.go` and its earlier grep, which had not searched for this pattern.
2. `grep` for the check-ID literals, `createNotDone` and `exhaustedRecoveryWayForward(` outside tests and `planglyph`. My context lacked the full caller set for F1.
