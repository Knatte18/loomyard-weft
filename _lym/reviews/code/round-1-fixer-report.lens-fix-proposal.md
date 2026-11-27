```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposed changes that were carried out as proposed (fix-proposal lens).

1. **MEDIUM**: `internal/gatecli/only.go`, the `markedScope` comment "so a pass always means every marked test ran" (F1), and the matching sentence in `pattern/PATTERN-test-speed.md`, "refuses a call that would leave a marked test out or run none".
   - The code does not hold this. `runTest` still applies `sc.guards = withoutHeld(sc.guards, request.held)` to a marked scope, after `markedScope`'s two refusals have already passed.
   - `withoutHeld` matches by bare name. Take `TestX` marked `upkeep` in package a and another `TestX` marked `finalize` in package b. `request.held` holds `TestX`, so the marked one is dropped from the guards.
   - If it was the only marked test, `--gate upkeep` runs nothing and exits 0, logging only "no guard test is left". That is the silent pass F1 set out to remove.
   - A marked scope's guards come from `request.marked`, which `withOnlyTests` already filtered by key per directory, so skipping `withoutHeld` for `scopeMarked` closes this.
   - A caller's own `-- -skip` can also leave a marked test unrun, so "always" overstates in that case too.

2. **LOW**: `internal/gatecli/only.go`, the new "would run no test" refusal (F1).
   - Its way forward says to set `gate.tiers.tier0.dir`, but names neither `lym config gate` nor `roleClause`. A non-prime agent is therefore handed a config step it may not take.
   - The sibling unlisted-tier refusal goes through `configRefusal`, and `gatecli/doc.go` says config-fix refusals close with the clause for a session that is not the prime.
   - In the no-`go.mod` case, "mark a test of the module" is also not a step that can be carried out there.

For F2 to F11 I built no case where the proposed change is wrong or breaks something elsewhere.

Extra tool calls: none.
