```yaml
lens: existing-paths
outcome: reported
```

I reviewed the record through the existing-paths lens, tracing each decision's state against the code paths that already write, reset, resume or read it.

**Paths traced, per decision**

- D1, `entry_gate` evaluation (`shedengine/entry.go` `enter`): when Drift-Baseline is skipped it routes by `on_done` to Parts-Write, and the skips chain through Parts-Plan-Write to Plan-Write. Survives.
- D1, goto (`shedengine/goto.go:123`, which sets `EnteredRow`): only the target's gate is bypassed. Survives.
- D1, `settledSegmentGuard` and `segmentWriter` (`loomcli/settledguard.go:97`): today it picks the row whose `on_done` is the bouncer, which is always Plan-Write. D1 replaces this with a history read. Survives once D1 is built.
- D1, rename (`shedengine/rename.go`) and the amendment path of `decision add` (`hasPlan` reads the `PlanWrite` engine, which a split recipe still holds). Survives.
- D3/D4, plan-dir naming (`planparser/parse.go:44`, which builds the path from `lymdirs.LymDirName`): `_lym/parts/` can be named the same way. Survives.
- D4/D6, the fresh path of `MultiLLMProducer.Call`: `prepareFreshSpawn` runs after the probe and before `archiveStaleOutputs`, so the parts archive runs first and the per-output archive finds nothing left. Survives.
- D5, `seatSpec` QuietHold (`prompt.go:171`, `!isChair && !seat.Required`): for writers to hold quietly the rule has to change, and D5 says so. Survives.
- D5/D6, failure of a Required seat: `stepEnder` → `join` → `mapOutcome`. **Breaks** (F1).
- D5/D9/D11, chair hold deferral after done (`work.go` `outstanding`). **Breaks** (F2).
- D5/D6, resume when the chair is not live: `Probe` → the fresh path → D6's archive. **Breaks** (F3).
- D6/D9, the chair's `plan` gate and the plan-lenses forked gate (`shedrecipe/entries_planwrite.go:83`): the forked gate is chosen only for a lead with fork authorization, and the chair forks none. Survives.
- D8/D9, readers of the plan directory after Done: Plan-Bouncer's `artifact_paths: _lym/plan` (`tapestry-recipe.yaml:252`) is static and does not read the chair's `join.md` pointer. Survives.
- D13, `reportengine/timingspans.go` `rowFamily`: "Parts-Write" and "Parts-Plan-Write" fall in family "Parts", which no table maps, and D13 adds them. Survives once D13 is built.
- `AddedDecisionTexts` readers (websterengine, beamwiring): Plan-Write's prompt doesn't use it, so the new rows lose nothing. Survives.

**Findings**

F1. **BLOCKING**, class `design`, D6 ("A Stuck from a failed writer or an exhausted gate halts the run (D1)") and D1 ("a Stuck halts the run as `Plan-Write`'s does").
A writer that fails, or whose gate is exhausted, does not produce a Stuck.
- A Required seat's failed end fires the step ender.
- `join` then stops the chair (`seatengine/engine.go:361-372`), and the chair's Wait ends with the strand gone.
- `mapOutcome` maps that to a returned error (`shedadapters/multillm.go:169-183`) or the join returns a wait error, and `errorExit` returns an error.
- Nothing outside `seatengine` reads `EndedStep`; the grep for `EndedStep` turned up no consumer.

So the run persists as failed, not as halted with a Stuck:
- The halt reason names the chair's death, not the writer or its Accept or gate findings.
- The planWrite-style decorator commits only on a non-empty pointer (`loomshed/planwrite.go:70-72`), so the part directories stay uncommitted in a dirty tree.
- An `ErrAgentDied` can be classed as transient and retried automatically (`shedtransient/shedtransient.go:32`).

The record's claim about today's mapping is false, and no decision changes it. Either `mapOutcome` turns a step-ending seat into a Stuck whose reason carries the seat's findings, or the record states the failed outcome and its recovery.

F2. **MEDIUM**, class `design`, D5 ("The chair's hold deferral counts writer seats as it counts advisors") against the fix loop in D9 and D11.
`advisorWork.outstanding` skips every run that has ended (`seatengine/work.go:44`, `run.ended()`).
A writer that finished and is then asked to fix its part has an ended run, so it never counts.
While the chair waits for that fix, its turn end is held and not deferred, and the parent gets a held-turn notice on every fix round.
D5's statement covers writers that are still running, not the post-done loop D9 and D11 rely on.

F3. **MEDIUM**, class `design`, D6's bound ("a resume keeps every finished part (D5); a writer whose strand is lost before it finished ends the step, and the rerun starts every part over").
D5's keep-finished rule runs only inside `Resume`, and `Resume` needs a live chair (`seatengine/resume.go`, start of `Resume`).
When the chair is not live, for example after it dies, times out or is lost in an interruption, `Probe` stops every live writer and returns an empty table (`resume.go`, end of `Probe`).
`Call` then takes the fresh path, and D6's `prepareFreshSpawn` archives every part directory (`multillm.go`, the `Call` fresh branch).
So losing the chair alone discards every finished part, and the record doesn't say so.

**Tool calls**

1. `AddedDecisionTexts` callers and `planparser.PlanDir` / `lymdirs` naming: to check whether the new rows need added decisions and how `_lym/parts` would be named. My context had no callers or path builder for these.
2. `shedtransient` `Class` and the seatengine test for `EndedStep`: to check how the error of a step ended by a seat is classed. My context showed only the grep hit.
3. Plan-Bouncer's recipe config and the routing of `beginAmendment`: to check whether a plan reader depends on the writer row's pointer. My context lacked Plan-Bouncer's `artifact_paths`.
4. Plan-lenses gate selection in `shedrecipe` and `loomshed/gates.go`: to check whether the chair's `plan` gate would become the lens-review gate. My context lacked `entries_planwrite.go`.
