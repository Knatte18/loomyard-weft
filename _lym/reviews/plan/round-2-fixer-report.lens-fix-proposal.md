```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge F18's proposal, which the fixer carried out as proposed in card 14's `prReviewGotoAdmitted` bullet.

**[MEDIUM]** The `stopLiveSession` sentences in card 14, "`stopLiveSession` resolves the runner inside the closure at call time… builds the pair's runner the way `wire` does otherwise".

- **Case:** under `lym loom goto --to PR-Review`, a record that names a strand makes `stepStopRunner` load the reed and shuttle configs, the reed geometry and the shuttle engines. That happens inside a verb whose lightweight wiring exists so that a broken module config does not refuse it.
- **What goes wrong:** if any of those loads fails, the `Admitted` hook returns the error after the move is written. The card's way forward is to run `lym loom goto --to PR-Review` again, and that hits the same load failure every time, so the way forward cannot be carried out.
- **What's missing from the card:**
  - It doesn't say that this one goto now depends on module config.
  - It doesn't give a way forward that differs when the failure is a config load rather than a stop.
- **Evidence:**
  - `internal/loomcli/cli.go:318-321`: "Such a verb needs no module config… a broken module config must not refuse it", with `goto` listed at :325.
  - `internal/loomcli/looparm.go:179-196`: `stepStopRunner` calls `reedengine.LoadConfig`, `shuttleengine.LoadConfig`, `hubgeom.ReedGeometry` and `shuttleengines.New`, each of which can return an error.
  - Card 14's engine-hook bullet: the error text "names `lym loom goto --to PR-Review` as the way forward".

No other breakage found. The dropped liveness read is safe:
- `StopStrand` goes to `removeStrandIfPresent` for a strand with no run record, and that returns nil when the strand isn't tracked or isn't removable (`internal/shuttleengine/stop.go:150-158`, `:226-234`).
- `stopRecorded` leaves a terminal record as it is (`internal/shuttleengine/stop.go:165`, `:171`).
- The relaunch sites run under the full `wire`, so `c.runner` is used there unchanged.

Tool calls:
1. grep for callers of `stopOrphanedStepRuns` and `stopStepRuns` in `internal/loomcli`, to see whether a lightweight verb already loads config through `stepStopRunner`. My context showed only that function's body.
