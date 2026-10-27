```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

The change set is two comment-only edits, so there is no code path to run and no test to check.

- **`internal/reportengine/doc.go:131`:** The sentence now lists Drift-Await in darn's setup phase. `darnPhases` at `internal/reportengine/timingspans.go:106` maps `loomshed.NameDriftAwait` to `PhaseSetup`, so the doc and the table agree.
- **`contracts/recipes/darn-recipe.yaml:51`:** The header claims such a run "has no awaited_at" and is "settled by hand with lym drift waive". The `awaited_at` field exists (`internal/driftengine/await_test.go`). The `waive` verb exists (`waiveCmd()` registered in `internal/driftcli/cli.go:47`). I did not check the "halts at Drift-Verify with no counted baseline" part against the code. It rests on the review's own account of `CountedCapture`.

Tool calls made beyond the context:
1. `grep` for Drift-Await/Drift-Baseline in `timingspans.go`, `awaited_at` in Go sources, and files containing `"waive"`. I needed to confirm the mapping and the field and verb the new text names. The first grep missed the mapping because the file uses `loomshed.Name…` constants.
2. `grep` of `darnPhases` in `timingspans.go` and of `waive` in `internal/driftcli/cli.go`. I needed this to confirm the `PhaseSetup` mapping and the `waive` command.
