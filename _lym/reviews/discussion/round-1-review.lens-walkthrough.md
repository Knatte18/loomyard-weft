```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's stated purpose.

**Decisions walked:**
- **D1:** webster-style spawn, wait and stop, with the store supplied by the caller and a runproc registration like `RoleSession`'s pane registration. Purpose met.
- **D2, plain path:** Darn-Write. `clean` passes, then `verify` fails on a test. The wrap spawns the fixer, the fixer commits `fix:` commits, `verify` is re-run, then `description` runs. Purpose met.
- **D2, restart path:** Darn-Write. The driver restarts while a fixer is mid-edit, with the tree dirty, and the `Expired` probe finalizes. Purpose broken (F1).
- **D2, Plan-Write forked:** the lens-review gate fails only for a missing lens-review record. The fixer is spawned but cannot fix it within its edit scope. Purpose met with waste (F3).
- **D3:** a recipe element names `darn.gate_fixer`, which resolves through `Env.Setting` at build. Purpose met.
- **D4, hand-off:** the fork's capped `--gate card` call goes to the detached runner, which runs it red and must decide whether to hand off. Input mismatch (F2).
- **D4, join:** the fork re-calls after the fixer's edits. Purpose broken once the re-run has ended (F4).
- **D5:** Discussion-Write fixer, with its edit scope and `validate-discussion` re-check. Purpose met.
- **D6:** `fix-verify` on the shared core and frame. Purpose met.
- **D7:** Discussion writer with the fixer on: it writes, runs the lens review, ends its turn, and the gate fails into the fixer. Purpose met.
- **D8:** the row fixer's record sits in run scratch and is read by D2's `Expired` probe. Purpose met, apart from the restart break in F1.

**Findings**

**F1 — BLOCKING, design, D2 (restart path) and the Constraints' first fact.** On a restarted driver the finalize evaluation never reaches the wrapped entry while the fixer has uncommitted edits.
- The `Expired` probe finalizes, and `evaluateGate` runs entries from the first one. It marks every entry after the first failure "not reached".
- On Darn-Write the first entry is `clean`, which fails on any uncommitted or untracked path. So a live fixer's in-progress edits fail `clean`, and the wrapped `verify` closure never runs.
- The finalize evaluation therefore never attaches to or waits on the fixer. It halts gate-failed with `clean` findings over the fixer's half-done edits.
- This is the same hazard the record cites to reject a pending result, now reached through D2's own restart path. The acceptance criterion "a run attached while a fixer record is held finalizes, waits on that fixer…" cannot hold on Darn-Write.
- Evidence:
  - `internal/shuttleengine/wait.go` `evaluateGate`: the `failedAt >= 0` branch reports not reached; the loop runs over `run.gate` from index 0.
  - `internal/loomshed/cleangate.go:16-27`
  - `contracts/recipes/darn-recipe.yaml:105-111`, where the order is `clean`, `verify`, `description`.

**F2 — MEDIUM, design, D4 Bound ("only a call from an agent strand hands off"; a fixer strand never hands off).** The runner that D4 makes decide the hand-off is never told who called.
- The front strips `agentname.StrandNameEnv` from the runner's environment.
- The request it writes holds only `Verb`, `Args` and `Cwd`.
- The fixer's call has the same verb, words and cwd as the fork's. Only its detach key differs.
- So D4 rests on caller identity that the step before it does not hand over, and the record does not say how that identity reaches the runner.
- Evidence: `internal/gatecli/detachfront.go:130` (environment strip), `:784` (`detachRequest{Verb, Args, Cwd}`), `:232-235`.

**F3 — LOW, design, D2/D5 on Plan-Write's `Forked` closure.** A lens-review gate failure that only a lens review clears is still handed to the fixer.
- `NewLensReviewPlanGate` adds a refusal for a writer that never recorded its lens review.
- D5's fixer is confined to `_lym/plan/` and re-checks with `validate-plan`, which does not run that check. It cannot clear that failure, so each such failure spends one fixer spawn before the writer is re-prompted.
- D5's leave-list names nothing that covers it.
- Evidence: `internal/loomshed/gates.go:292-301`.

**F4 — MEDIUM, design, D4 step 3 (join "while the run is in its fixer or re-run phase").** After the re-run ends, the fork's next call no longer joins and starts a fresh run.
- Once the re-run has written its exit, the run is out of those phases. `joinable` then applies the tree-identity rule against the tree recorded before the fixer.
- If the fixer changed HEAD's tree or the dirty-path set, a fork call that arrives after the re-run finished (between `done: false` answers) is not joinable.
- It starts a fresh run. That run drops the fixer's reply and, if red, spawns a second fixer, which breaks "at most one fixer per red call".
- Evidence: `internal/gatecli/detachfront.go:316-326`. The tree check precedes the `hasExit` case.

**Tool calls**
- Read `evaluateGate`'s body in `wait.go`, because the context held only grep lines and I needed its order and stop-at-first-failure rule.
- Grepped for the Darn-Write clean gate to find the file that implements it.
- Read `writeNewRun` and the request types in `gatecli`, because the context lacked what the runner is handed.
- Read `cleangate.go` to learn what `clean` checks.
- Read `NewLensReviewPlanGate` and `DiscussionTable`, because the context lacked the forked plan gate's extra check and the discussion prompt's inputs.
