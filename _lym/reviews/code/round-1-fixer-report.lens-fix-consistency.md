```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens on the F1–F3 change set (`darnwiring.go`, `wiring_darn_test.go`).

Findings:

- **LOW** — `internal/loomshed/darnwrite.go:47`, the `DarnDeps.LatestOutcome` doc comment.
  - It says "returns the latest Darn-Write history entry of the run; found is false when the row has no entry yet."
  - The one implementation, `latestDarnOutcome` in `darnwiring.go`, now passes over reroute, goto and skipped entries, as its own doc comment says.
  - A caller reading only the seam's contract would expect a goto or skipped entry to count.
  - The reroute skip was already missing from that wording before this fix, so the fix widened an existing gap and did not open a new one.
  - Suggested change: say "latest Darn-Write entry that records a spawn", and "no such entry yet" for found.

Otherwise the change set agrees with its surroundings:
- The `latestDarnOutcome` doc comment now names the skipped kinds, and the code skips exactly reroute, goto and skipped.
- The new `TestLatestDarnOutcome` rows use the existing `firstNonEmptyState` helper and the `wantFound` zero-value convention.
- The new `underFileDir` and `wantErr` fields are commented in the same style as their siblings.
- The `TestDarnDeps_LatestOutcomeNamesTheRowsFindingsFile` test uses `shedrun.GateFindingsFile`, the function `Env.GateFindingsFile` resolves to, as `TestWire_DarnRecipe` already pins.

Tool calls (1 of 6):
1. `grep` over `internal/loomshed/*.go` for the outcome names and `LatestOutcome`. My context did not hold the seam's declaration or any other doc that describes the skipped kinds.
