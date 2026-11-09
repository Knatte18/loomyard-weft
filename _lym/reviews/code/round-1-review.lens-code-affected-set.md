```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens. Findings first, then the per-change set, then my tool calls.

**Findings**

1. **MEDIUM, consistency, `internal/shuttleengine/wait.go` (hunk at the old line ~2048, `finalizeAndRemoveDiedWindow`)**
   - The new `gateEndedEarly` method was inserted between `finalizeAndRemoveDiedWindow`'s doc comment and its `func` line.
   - The diff shows the comment `// finalizeAndRemoveDiedWindow finalizes the run with outcome, then ends the window…` directly above `// gateEndedEarly returns why…`.
   - As a result `gateEndedEarly` carries two stacked comments, and `finalizeAndRemoveDiedWindow` has none.
   - Fix: move `gateEndedEarly` and its comment above, or below, the `finalizeAndRemoveDiedWindow` comment and function pair.

2. **LOW, scope, `internal/gatecli/cli.go` (the `Long` text, around line 85)**
   - The diff removes the help line `-C names the run directory, with explicit targets only.` from `test`'s `Long`.
   - The `-C/--dir` flag still exists at `cli.go:118`, so the help text drops a live option with no licence in the decision record.
   - Fix: restore the line.

**Per change: the set I found, and whether it is covered**

- **`NewVerifyGate` gained a `budget` argument.**
  - The only production caller is `shedrecipe/entries_gate.go:276`, which the diff updates.
  - `websterengine.NewVerifyGate` is a different function.
  - The test callers are updated in the diff.
  - Covered.
- **`Env.VerifyBudget` is a new field.**
  - `envkit.nilLegal` gains `VerifyBudget.Notify`.
  - The darn wiring sets it.
  - Every other run leaves it zero, which turns the budget off.
  - Covered.
- **New config key `darn.verify_budget_min`.**
  - `Config`, `ConfigSpec` (`CategoryNone`), `template.yaml`, `darnengine/doc.go` and the recipe comment are updated.
  - `configreg_test` and `darnengine/config_test` are updated.
  - `verify_attempts` also appears in `loomrecipe/darnrecipe.go`, `loomrecipe/doc.go`, `hubreconcile` and `configsync` tests.
  - Only the recipe-builder stamping is affected, and a new key needs no reconcile since it is not a meaning change.
  - I found no stale list of darn keys.
  - Covered.
- **`lym gate test --stop-on-fail`.**
  - `gatecli/cli.go`, `gatecli/doc.go` and `run.go` are updated.
  - The `--gate`/`--upto` default of running every tier is kept.
  - `docs/overview.md` and `docs/dotnet-hub.md` mention `--guards`.
  - I did not check them for a flag list, and I saw no sign that they enumerate flags.
  - Covered, apart from finding 2.
- **`FailureReportHeader` moved into `gateslot`.**
  - `gatecli/failurereport.go` now takes its header from `gateslot`.
  - The only other consumer is the new `FailedTests`.
  - Covered.
- **`gateFailedReason` wording and `GateOutcome.EndedEarly`.**
  - `gate.go`, `run.go`, `wait.go`, `shuttleengine/doc.go`, `shedadapters/gateattempts.go` and the tests are updated.
  - The only other reader of `GateOutcome.Attempts` wording is `gateFailedReason`.
  - Covered, apart from finding 1.
- **PATTERN.**
  - `PATTERN-gate-slot.md`'s Verified tree section is updated for `--stop-on-fail` and `FailedTestsCall`.
  - The focus text lists no declared PATTERN entries, so there is nothing further to check.
  - The darn rerun lives in the loomshed gate, and `gateslot.FailedTestsCall` is the one place that spells tiers and targets.
  - Covered.
- **Commit messages.**
  - None carries a `Pattern-Departure:` trailer, and none is needed.
  - Covered.

**Tool calls**
1. A grep for the callers of `NewVerifyGate`, for `verify_attempts` outside Go and `_lym`, for `--guards`, and for the "past a failed" wording. My context did not hold the full caller and doc sets.
2. A git grep of the old `gateEndedEarly` location, and a check that `-C names` was removed while the `-C` flag stayed. This confirmed finding 2. The `gateEndedEarly` grep hit the wrong file and returned nothing, so finding 1 rests on the diff text alone.
