```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every bound, guard and claim about existing code in `decision-record.md`.

**Claims attacked:**
- D1, bound "each row reads the flag on entry, a goto bypasses only its target's gate": held. I did not verify the goto mechanics; they are not in context.
- D1, the settled-verdict refusal names the writer from `Plan-Write`'s latest history entry: held for every flag-flip path I built.
- D3, the `archive-` prefix refusal keeps D4 and D6 archiving consistent: held.
- D3, the shared-spot ordering rule ("no card position satisfies both"): held. A spot using the first needing part's own export is left to the chair, and Open risks already records that.
- D4, the fresh-spawn archive never nests and the commit pathspec covers both the moves and the archive: held.
- D5, a done writer whose kept pane the probe removed counts as finished on resume (`resume.go` Probe doc, `probeSeat`): held, and the record names this case.
- D5, the zero `Result` passed to Accept on resume decides nothing: held, given the D7 check reads only disk.
- D5, the hold deferral counts a done writer mid-turn: held within one session. Every reinvoke removes done writers, and D9 covers that.
- D6, existing-code claims (`join` stops the chair on a required seat's end, `mapOutcome` never reads `EndedStep`, `planWrite.Call` commits only on a non-empty pointer): held (`engine.go` `join`; `multillm.go` `mapOutcome`; `planwrite.go:73-93`).
- D6, "a writer's exhausted gate or failed Accept leaves the run Stuck": broken by a chair timeout (finding 2).
- D6, the writer outcome classes (Stuck vs error): broken for a writer timeout (finding 3).
- D6, Bound "chair not live → probe stops every writer, fresh path archives every part" (`resume.go` Probe; `multillm.go` Call): held.
- D8, Bound "overwrites only while `Parts-Plan-Write` is the current row, so a run past that row cannot reach it": broken (finding 1).
- D8, "within the row a rerun discards only what the chair added, which D9 keeps the chair from doing": broken, because only the chair is restrained (finding 1).

**Findings:**

1. MEDIUM — design — D8, Bound and the current-row refusal.
   - **Defect:** a current-row check does not bound `join-parts` to a live `Parts-Plan-Write` step.
   - **Halted states keep the row current:** a run halted Stuck (Blocked), Failed or Paused keeps `CurrentProducer` at `Parts-Plan-Write`. So while a run is halted there, any agent can run `join-parts`: the stop chain's tackler or arbiter, orch, or the operator. It rewrites the committed gate-failed joined plan from the part directories, drops the chair's shared-spot cards and framing, and leaves a dirty tree.
   - **Goto counterexample:** a run halted at a later row (a Plan-Review row) is moved by `lym loom goto --to Parts-Plan-Write`, which D8's own way forward recommends. Before `resume`, the current row is already `Parts-Plan-Write` and the part directories are not yet archived, so `join-parts` succeeds. It overwrites the reviewed plan uncommitted. The fresh spawn's `NewPlanDirRotator` then archives the overwrite and builds the chair's prior-plan amendment from it instead of from the reviewed plan.
   - **Writers are not bound:** within the row, a part writer running `join-parts` after the chair's shared-spot cards are in discards them. D9 restrains only the chair, and D11 forbids writers only to "edit the plan directory", not to run the verb.
   - **Fix:** also require the run state Running, or the caller to be the chair's strand, and state which callers the bound admits.
   - Evidence: `internal/shedengine/run.go:402`, `:416`, `:432` (Failed and Blocked persisted with `st.CurrentProducer`); `run.go:258`, `:383` (Paused likewise); `decision-record.md` D8 lines 178-179, D9 line 190, D11 line 219.

2. MEDIUM — design — D2 ("Both rows take `plan_timeout_min`") against D6 ("a failed writer… leaves the run Stuck").
   - **Defect:** the chair gets the same timeout as each writer, but its work spans the writers' whole time plus fix rounds, the join and the shared spots, and it starts before them (`Run` starts the chair first).
   - **Counterexample:** a slow writer uses most of its budget. The chair times out first, `chairDone` wins `join`'s select, `stepEnded` stays false and `EndedStep` is never set. `mapOutcome` takes the chair-timeout branch and returns an error, not Stuck. Nothing is committed, and the reason names the chair, not the writer.
   - The same race arises when a writer's failure and the chair's end land together, since a chair that ended "decides the step as usual".
   - **Fix:** the record must give the chair a timeout that covers its writers, or state what this outcome is.
   - Evidence: `internal/seatengine/engine.go` `Run` (chair started first), `join` (inner select prefers `chairDone`; `markEnded` only when `stepEnded`); `internal/shedadapters/multillm.go` `mapOutcome` (`OutcomeTimeout` → error); `decision-record.md` lines 62, 142-146.

3. LOW — design — D6, writer outcome classes.
   - **Defect:** a writer that times out matches neither of D6's classes. Stuck covers "ended done with a failed gate or an Accept error, or not found at resume". Error covers "failed to start or died".
   - **Why it matters:** `watch` ends the step for any non-finished end of a required seat, `OutcomeTimeout` included, so the mapping must decide timeouts. As written it is unspecified whether they halt committed (Stuck) or fail uncommitted (error), and whether they are retried as transient.
   - Evidence: `internal/seatengine/engine.go` `watch` (`finished` requires `OutcomeDone`; otherwise `ender.end(r)`); `decision-record.md` lines 142-144.

Tool calls:
1. `internal/loomcli/settledguard.go` lines 80-140 and recipe segment grep: to check `segmentWriter` against D1's refusal claim; the manifest read only the top of the file.
2. `internal/seatengine/engine.go` `Run`/`startSeat` and `table.go` `leadName`/`ValidateSeatName`: to check the required-seat start-failure path and part-name collisions; the manifest held only `join` and `watch`.
3. grep of `internal/loomrecipe` for row constants: to find the tapestry rows' review segments; it found only test files.
4. grep of `internal/shuttleengine` for `Stop`: to learn what a stopped chair's `Wait` reports.
5. `internal/shuttleengine/stop.go` and `wait.go`: to confirm a stopped chair finalizes as stopped or died rather than as a wait error, which bears on D6's mapping claim.
6. grep of `internal/loomengine/plan.go` for `Segment`: an inconclusive check of whether the new rows would sit in the plan review segment, so I raise no finding on it.
