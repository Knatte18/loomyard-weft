```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens, checking whether each guard, refusal, branch and test row the plan adds can be reached, and whether the site it guards comes back somewhere unguarded.

**Per guard: reachable or not**

- Card 1, every `LoadConfig` and `Step` refusal: reachable (hub-file rows and a local-layer row).
- Card 2, `TakeRun` refusing at the budget: reachable through the agent's repeated `lyx drift compare` and `capture --side` runs.
- Card 5, `RunCapture` lock refusal after `LockRetry`: reachable when two starts race past `StartCapture`'s probe.
- Card 5, `RunCapture` returning `Check`'s refusal with no record written: reachable when the tree or HEAD changes between the spawn and the child's check under the lock.
- Card 5, `CheckMainCapture`:
  - wrong-row refusal: reachable through `lyx drift capture` at any other row, or at `Drift-Await` while it waits.
  - moved-HEAD refusal: reachable after an adopt or a code commit.
  - dirty-tree refusal: reachable when the operator leaves tracked changes.
  - the "also admits a capture-halt `blocked` at `Drift-Baseline`" clause: always true already, because the first clause admits `Drift-Baseline` in any state. Redundant, not a defect.
- Card 5, `StartCapture`:
  - refusal on a lock held before the start: reachable through `lyx drift capture`. On the row's path it is effectively unreachable, because card 9 checks the lock first. Fine.
  - breakaway refused, with and without `FallbackDetach`: reachable through the seam.
  - child exits before its record: reachable.
  - "a start the bound outlasts": reachable, but no `TestStartCapture` row reaches it (finding 3).
- Card 6, `RunCompare` refusing on a held marker: reachable when the agent runs two compares at once.
- Card 7:
  - `RecordScope`'s refusals (wrong row, second call, bad `--skip`/scope pair): reachable; a re-entered agent hits the second-call refusal.
  - `Withdraw` of a test not on `accepted`: reachable.
  - `Defer`:
    - wrong-row or wrong-state refusal: reachable.
    - refusal while the capture lock is held: reachable after `lyx drift capture` at either halt.
  - `Waive` with a counted capture: reachable at a `Drift-Verify` halt on a failing compare.
  - `CheckVerifyRow`: reachable.
  - `ErrRunsSpent` wording for the agent and for a caller: reachable. Test reach is the way-forward fork's question.
  - `Accept`:
    - refusal in a state other than `running` or `blocked` (failed, paused, awaiting): reachable.
    - the agent-versus-caller split: the guarded state recurs (finding 4).
- Card 9:
  - agent ends Done with no scope record: reachable.
  - `awaited_at` set, then Done: reachable via `lyx drift defer` at a capture halt, then resume.
  - held lock with `WaitCapture` (wait) and without it (Done): reachable.
  - `running` record with a free lock: reachable after the child dies.
  - `ErrBreakawayRefused` fallback: reachable through the seam.
  - another `StartCapture` error: reachable.
  - undeclared capture: reachable.
  - dirty tree and moved HEAD: reachable.
- Card 10:
  - missing record under `partial` or `full`: reachable (adopt, or a `StartCapture` error that was resumed).
  - `failed` record, timed-out record, `running` record with a free lock: reachable.
  - waiting on a held lock: reachable.
  - `passed` with HEAD moved: reachable (an operator commit, or a Plan fixer under overlay).
  - `awaited_at` set, then Done: reachable after a Webster reroute.
- Card 11:
  - steps 1, 3, 4, 5 and 7–9: reachable. Step 3 is reached by an accept followed by an error stop; step 4 by the flag-off-and-on sequence or an adopt.
  - step 2 resume detection: reachable, since a resume appends no history entry, so the Stuck stays last (`internal/shedengine/status.go:142`, `PendingBy`).
  - step 6: reachable, but it fires on states it is not meant for (finding 2).
  - the mechanical steps on a reinvoked entry: the guarded site recurs unguarded (finding 1).
- Card 12, activity rows, including a lock path occupied by a directory: reachable. gofrs/flock opens with `O_CREATE`, and that fails on a directory (`gofrs/flock@v0.12.1/flock.go:64`).
- Card 13, the capture-kill error row: reachable through the seam.
- Card 14:
  - entry refusals: reachable.
  - entry-gate rows, including `EnteredRow` naming `Drift-Await`: reachable.

**Findings**

1. **BLOCKING, design.** Card 11 `11-drift-verify-producer.md` steps 1–7, together with card 14's `InterruptPolicies` entry for `Drift-Verify` (Reinvoke).
   - The "no fix goes uncompared / the agent runs alone" guarantee is enforced only inside the seat's gate. The re-step path a Reinvoke policy permits does not guard it.
   - When a step is interrupted while the verify agent's chair is still live, the re-step calls `DriftVerify.Call`. Its dirty-tree check (step 6) or its mechanical compare (step 7) then runs before `NewMultiLLMProducerSourced`'s live-chair probe is ever reached.
   - That gives either a dirty-tree Stuck that orphans the live agent, or a passing compare that returns Done while the agent keeps editing. Either way, code lands uncompared.
   - Card 14's justification is that the agent rows' seat producer "probes for a live chair and reattaches". That does not hold for this row.
   - Evidence: `internal/loomshed/interruptpolicy.go:16-19` (the Reinvoke premise is the adapter probing before anything else); `internal/shedadapters/multillm.go:84-99` (the probe happens inside the producer's `Call`); card 11's step order.
2. **MEDIUM, design.** Card 11 step 6.
   - The step re-runs the dirty and verify checks whenever HEAD differs from `AttemptHead`. That state also occurs on a first entry, where `AttemptHead` reports found false and the card does not say what happens.
   - It also occurs on a loom Finalize reroute, where Parent-Align's merge and the Code perch's fixes move HEAD. The `drift-verify` gate then runs over commits no `Drift-Verify` session made, and a failure Stucks the row, telling the reader to "fix the tests".
   - D6 licenses the gate only "when the session committed code".
   - Evidence: card 11 step 6 and its test row "an entry with HEAD past the attempt head and a failing verify gate is Stuck before any compare"; `contracts/recipes/loom-recipe.yaml:79` (Finalize reroutes to Parent-Align); decision record D6, lines 191 and 57.
3. **LOW, consistency.** Card 5, `StartCapture`.
   - The branch "a start the bound outlasts", where the record-wait bound expires, is an error path. No `TestStartCapture` row reaches it.
   - Evidence: card 5's `StartCapture` paragraph against its test list.
4. **LOW, design.** Card 7, `Accept`, with the Shared Decision "run status decides the caller".
   - The status is also `running` while the row's own mechanical compare runs, and through the dirty, verify and compare entries before the agent spawns. An operator's or orch's `lyx drift accept` in that window is classified as the agent session's.
   - Under `full`, that gives an `unruled` acceptance with `by: agent`, so the parent's own accept is recorded as the agent's. The fail-safe still holds: it halts for a ruling.
   - Evidence: `internal/shedengine/status.go:18` (state constants); card 7's `Accept` paragraph.

Tool calls:
- `go env GOMODCACHE` plus a grep of gofrs/flock's `OpenFile` flags, to check whether card 12's "lock path occupied by a directory" error row can be reached. The manifest held no flock source.
