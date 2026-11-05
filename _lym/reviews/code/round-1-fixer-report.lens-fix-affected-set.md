```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1's set is the zero-`TurnEndedAt` cases that `doc.go:185` names: a pane-decided reading and an interrupted woken turn. Both reach `woken` through the same `!TurnEndedAt.After(mark.at)` expression at `deliver.go:287`. The new row sets `time.Time{}`, which exercises that expression, so one row covers the set.
- F2's set is the sites that describe the hold. These are the test doc comment, `deliver.go:25`, `deliver.go:256`, `doc.go:177` and `doc.go:185`. Only the test doc comment lagged, and the fix updated it. The rest already state the hold.
- Not checked: I did not run the new row against a mutated `deliver.go` to confirm it fails with the defect in place. That belongs to the correctness lens.

Tool calls:
1. `grep` over `internal/creelengine` for `ringAnswerWindow`, "no turn end" and `TurnEndedAt`: to list every site describing or branching on the hold, which my context lacked outside `deliver.go:235-314`.
