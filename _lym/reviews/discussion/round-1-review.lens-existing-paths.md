```yaml
lens: existing-paths
outcome: reported
```

Task: the existing-paths lens over the cold-gate-fixer decision record, listing traced paths per decision and the findings they produce.

**Paths traced (survives or breaks)**

- D1. Webster's `RemoveRecoveryStrands` stops `VerifyFixers` strands (`internal/websterengine/strand.go:134`). **Survives**, because D6 keeps those records as the store.
- D1. `planfixer.go:201` calls `awaitColdFixer`. **Survives**, because D6 moves the wait for the other cold strands too.
- D1. `runproc` pane registration has a precedent: `RoleSession` via `registerPane` (`internal/loomcli/sharedbootstrap.go:333-335`). **Survives.**
- D2. Darn-Write `Reprompt` is set after `resolveGateSpec` (`internal/shedrecipe/entries_darnwrite.go:29`). **Survives**, because the wrap is applied after the constructor.
- D2. Plan-Write `Forked` is set after the resolver (`entries_planwrite.go:83`). **Survives** for the same reason.
- D2. `probeExpiry` skips entries with `Attempts <= 0` (`internal/shuttleengine/wait.go:479`). **Survives.**
- D2/D8. The row fixer record lives in the run-wide scratch directory (`internal/loomcli/wiring.go:627`, `internal/shedrun/paths.go:155`). No goto, reset or row re-entry clears it. **Breaks**: see finding 3.
- D2/D5. Plan-Write's forked gate runs a presence check on `lens-review.md`, shared with `validate-plan --lens-review` (`internal/loomshed/doc.go:17`). **Breaks**: see finding 2.
- D3. The recipe gate element key set is closed (`gateEntryKeys`, `internal/shedrecipe/entries_gate.go:20`). **Survives**: D3 declares the new key.
- D3. gate.yaml's `gates` are decoded as `yaml.Node` per name (`internal/gateslot/config.go:76`). **Survives.**
- D4. The front strips `StrandNameEnv` from the runner's environment (`internal/gatecli/detachfront.go:130`). **Survives**: the front, which still sees the strand name, decides the hand-off and the key.
- D4. `pruneOldRuns` and `delivered` in another key directory do not touch the waiting run, since the fixer's key includes its strand name. **Survives.**
- D4. The runner is registered as `RoleGate` (`detachfront.go:816`). **Survives**, as long as D1's runproc registration also covers the agent-call fixer.
- D7. The Discussion twill directive (`internal/loomengine/twill.go:96,103-104`) is named by D7. **Survives.**
- D7. The Plan-Write twill directive (`internal/loomengine/twill.go:22,66-67`) is not named by D7. **Breaks**: see finding 1.

**Findings**

1. **MEDIUM — design — D7.** The Plan-Write lens procedure keeps its validator loop.
   - The plan twill directive tells the writer to run its lens review only once `lym loom validate-plan` passes.
   - It also carries `SelfCheckCommand: "lym loom validate-plan"` and `FinalSelfCheckCommand: "lym loom validate-plan --lens-review"`.
   - D7 drops only the Discussion fork boilerplate and the Discussion lens procedure's closing loop.
   - Plan-Write runs the lens review by default (`plan_lenses`). On that default path the writer still loops on its validator, so the plan gate never fails and the fixer never runs, which is the outcome D7's rationale says the change prevents.
   - Evidence: `internal/loomengine/twill.go:22`, `:66-67`; `internal/twilldirective/twilldirective.go:57-60`.

2. **MEDIUM — design — D5/D2.** The Plan-Write fixer can satisfy the forked gate's `lens-review.md` presence check by writing that file.
   - The forked plan gate adds a presence check on the recorded `lens-review.md`.
   - D5 limits the Plan-Write fixer to "only the files under `_lym/plan/`". Its leave-for-the-writer list names decisions, criteria, PATTERN changes, cards and test assertions, but not the lens-review record.
   - So a fixer can write that record, and Go's re-run passes the presence check with no lens review done.
   - The record does not say that a missing lens review is left for the writer or kept out of the fixer's scope.
   - I did not read where `lensReviewPath` points; I infer it is inside the plan directory from `DispositionPlace`.
   - Evidence: `internal/loomshed/doc.go:17`; `internal/loomengine/twill.go:67-68`.

3. **MEDIUM — design — D2/D8.** The `Expired` probe can read a terminal record left by an earlier visit of the row.
   - The row fixer record lives in `shedrun.ScratchDir(location, runID)`. That directory covers the whole run, is shared by every visit of the row, and nothing clears it on goto, reset or re-entry.
   - D2's probe "answers true when the store holds a fixer record from before an attach".
   - A record from an earlier visit therefore finalizes a fresh attach at once, and the finalize evaluation attaches to a fixer that has already ended.
   - Webster's precedent attaches only to a last record whose `Status == ""` (`internal/websterengine/verifyfixer.go:81`).
   - D2 and D8 state neither that a terminal record is cleared or ignored nor that a new visit starts with an empty store.
   - Evidence: `internal/loomcli/wiring.go:627`; `internal/shedrun/paths.go:155`.

**Tool calls**

1. Grep for `RunScratchDir` resets and `verify_attempts` stamping: I needed to know who resets the row scratch directory and whether stamping rewrites the verify element.
2. Grep in `shedrun`, `shedverbs` and `shedengine` for scratch-dir clearing: needed for finding 3's path.
3. Grep for where `RunScratchDir` is set and any goto or reset clearing: confirms the scratch directory covers the whole run.
4. Grep of gateslot `GateKey` fields, recipe element keys and the writer self-check rendering: the gate-key schema and the D7 rendering path were not in the reading I had.
5. Read of the twill.go spawn points and self-check rows, plus the `lens-review` presence check: needed for findings 1 and 2.
