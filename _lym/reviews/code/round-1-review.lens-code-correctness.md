```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `darn-drift-await-row` diff. Findings: none.

Per function or branch, with the case I ran:

- `DriftBaseline.Call`, seam check: the `Sleep` seam is gone and the others are unchanged. A nil `Sleep` no longer fails the pass, which is right because the pass no longer reads it. Correct.
- `DriftBaseline.Call`, `var commitAll bool`: it starts false, and the detached-start path returns before `captureAgain` sets it. Only the in-process path sets it, so a detached start commits just the scope record and `baseline.md`. This matches the doc. Correct.
- `DriftBaseline.drive`, lock held (`held == true`): it returns `Done` with `done` and does not wait. Correct, and the acceptance criterion says the same.
- `DriftBaseline.drive`, lock free and capture passed: it returns `Done`. Correct.
- `DriftBaseline.drive`, lock free and no capture or a failed one: it goes to `captureAgain`. Correct.
- `captureAgain`, `StartCapture` returns nil: `Done`. Correct.
- `captureAgain`, `StartCapture` returns a non-breakaway error: it returns a wrapped error. Correct.
- `captureAgain`, `StartCapture` returns `ErrBreakawayRefused`: it falls through to `*commitAll = true` and the in-process run, as the doc says. Correct.
- `CountedCapture`:
  - not found: false. Correct.
  - found but failed: false. Correct.
  - found and passed with `AwaitedAt == ""`: false. Correct.
  - found and passed with `AwaitedAt` set: true. Correct.
  - Callers `Waive` (`verbs.go:173`) and `DriftVerify.pass` (`verify.go:164`) are updated to the 3-argument signature. Both now apply the `awaited_at` rule on every recipe, as the decision record asks.
- `driftBaselineEntry`: `configRejectUnknown(cfg, "subject")` rejects a `capture` key, and the error appends `driftKeyWayForward`. The `Drift.Sleep` seam check is removed from this entry.
  - Drift-Await still needs `Sleep`. `await.go:56` calls `awaitCaptureEnd` with `d.Sleep`, so `Sleep` stays in `Deps`.
  - I did not check whether `driftAwaitEntry` itself still requires `Drift.Sleep`. It is unchanged by the diff.
- `darn-recipe.yaml`: the chain is Drift-Baseline → Drift-Await → Darn-Write, and Drift-Await carries `entry_gate: drift.enabled` and no config key. Correct, since DriftAwait rejects every key.
- `reportengine/timingspans.go` `darnPhases`: Drift-Await is mapped to `PhaseSetup`. Correct.
- `driftcli/wiring.go` `verbStatus` (focus item 1):
  - It reads status through `state.ReadJSON` and a not-found status returns the way-forward error.
  - The seed read and `AwaitRow` are gone, so loom and darn runs behave the same.
  - The `shedrun.ReadSeed` removal leaves no dangling use. `go build` and `go vet` of `driftcli` and `loomcli` were clean.
  - Stale comment: none. The doc line was updated.
- `loomcli/driftwiring.go` `driftDeps` and `loomcli/wiring.go` (focus item 2):
  - The `awaitRow` parameter and the `Deps.AwaitRow` assignment are gone, and the call site passes `driftTables` directly.
  - The parent's `ReadJSON` call at `driftwiring.go:61` sits beside these changes with no leftover.
- `segmentBounces` (`wiring.go:~933`): the comment "A run directory not yet created fails the lock open with not-exist" now sits above `state.ReadJSON`, which the parent swapped from `ReadJSONNoMkdir`. If `ReadJSON` creates the directory, the comment is stale. I treat that as the parent's own line, outside the task's change, so I record it only as a note and not a finding.
- Focus item 3: a repo-wide grep for `AwaitRow`, `WaitCapture`, `awaitRow` and `ReadJSONNoMkdir` found no matches outside `_lym`. No `capture: wait` remains either. The removal is complete after the merge.

Tool calls:
1. `git diff` of the core changed files (driftengine, shedrecipe, recipe yaml, reportengine). My inherited context held only the wiring diffs, not these.
2. `git grep awaitCaptureEnd` plus `sed -n 95,125p` of `baseline.go`. This confirmed `awaitCaptureEnd` still has a caller in `await.go` and let me read the `drive` flow around the held-lock branch.
