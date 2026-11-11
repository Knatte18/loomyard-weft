```yaml
lens: fix-affected-set
outcome: reported
```

Affected-set check for F1, the one finding with a set premise:

- **F1 (the refusal wording sites): no findings.** I searched `*.go` and `*.md` outside `_lym/` for the old wording. It appears only at `internal/boardcli/doc.go:115`, which the fix reworded, and `internal/boardcli/runsettings.go:99`. The `runsettings.go:99` message already says "which is no orch of the entry's beam", so it agrees with the reworded doc and needs no change. No other doc or spec carries the unqualified claim.

Unchecked findings: none.

Tool calls:
1. `grep` across `*.go` and `*.md` for the old refusal wording, to find sibling sites that my context did not hold.
