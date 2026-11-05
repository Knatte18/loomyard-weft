```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every site of F3's set (the repeated "done" literal as a board status in `internal/beamcli`). F2 and F4 have no set premise, so I left them unchecked.

No findings.

The one search was `grep '"done"'` over `internal/beamcli/`. The remaining hits are not board-status uses:
- `create_test.go:178` is an error-substring expectation.
- The `dobbypairanchor_test.go` lines are JSON fixtures with run-state and producer-outcome values.

Every board-status comparison in the package now uses `boardengine.IsDoneStatus` or `boardengine.StatusDone`.

Tool calls:
1. `grep -rn '"done"' internal/beamcli/`: my context held only the three files the fixer read, not the rest of the package.
