```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code that already runs against the state each decision adds or changes.

**Findings**

1. **BLOCKING — D5, D6 and D13: any interruption after the first part finishes ends the step.**
   - The record says resume keeps today's rule, and D6 bounds the damage to "a crash that loses any writer's strand". The probe makes it much broader.
   - Evidence: the seatengine doc's "Probe before archive" says a terminal advisor whose strand is still up, a done one kept by its pane included, "is removed by the probe itself … and answers not found". The Resume section says "a required advisor the probe did not find ends the step at once".
   - D5 makes writers required and keeps them in their panes after done. D13 gives both rows the reinvoke interrupt policy, which goes through Probe.
   - So a pause, hibernate or driver restart reinvokes the step, and every writer that already finished is removed and ends it. The row is Stuck and the run halts. A split run cannot survive an interruption once any part is done.
   - D5 needs its own resume rule for done writer seats: keep them, or treat their accepted output as standing.

2. **MEDIUM — D5 and D9: a resumed chair is told a finished writer "no longer answers".**
   - Evidence: the Resume section says a found advisor whose outputs all exist gets the line that "its output stands and the seat no longer answers".
   - D9 sends a contract mismatch back to that writer, and D5 says writers keep answering after done. After a resume the chair is told the opposite.
   - D5 does not say which line a writer seat gets on resume.

3. **MEDIUM — D6: the commit pathspec drops staged deletions.**
   - D6 commits "the plan directory and `_lym/parts/`". Today's plan commit uses `planCommitPathspec`, which adds webster's durable directory and the plan amendment record. The comment at `internal/loomcli/wiring.go:118-123` explains why.
   - D6 reuses `loomshed.NewPlanDirRotator`, which runs `ArchiveWebster`. A webster record moved by that rotation would leave its deletions unstaged with the record's pathspec, so the tree stays dirty after Done.

4. **MEDIUM — D1: the settled-verdict refusal points split runs at the solo writer.**
   - Evidence: `internal/loomcli/settledguard.go` `segmentWriter` returns "the first row in list order outside segment whose OnDone names bouncer". After D1 that is still `Plan-Write`; `Parts-Plan-Write`'s `on_done` is `Plan-Write`, not `Plan-Bouncer`.
   - In a split run, a goto onto a settled Plan-Review row tells the operator `"lym loom goto --to Plan-Write" re-runs the writer`. That silently swaps a split run to a solo plan.
   - D1 names that goto as a deliberate fallback but says nothing about this message reaching split runs.

5. **LOW — D4: the stale-parts archive may fall outside the committed pathspec.**
   - D4 archives a stale `_lym/parts/` "into an archive directory beside it" and commits only `_lym/parts/`. If the archive sits outside `_lym/parts/`, its new files are never staged and the moved files' deletions are left out of the commit.
   - Same class of problem as finding 3.

**Paths traced**
- D1:
  - Entry-gate skip chain (`shedengine.enter`): survives.
  - Goto bypassing the gate (`goto.go` sets EnteredRow): survives.
  - Entry-gate operand check (`SettingKnown` is `configreg.CheckSettingKey`, and the keys join the template in D2): survives.
  - Settled guard `segmentWriter`: misreads in split runs (finding 4).
- D2: profile and `run_settings` key check through configreg: survives.
- D4: Parts-Write commit versus where its archive goes: unclear (finding 5).
- D5 and D6:
  - Probe removing done seats, then Resume ending the step: breaks (finding 1).
  - Resume's "output stands" line: misreads (finding 2).
  - Hold deferral (ended seats never count): survives.
- D6:
  - Plan-dir rotator's ArchiveWebster against the commit pathspec: breaks (finding 3).
  - `PlanReviewSkippable` and rework `first_card`: survives. Rework generations come from the rework session, not from these rows. A joined plan with `first_card` 1 matches no round, as a goto Plan-Write plan does today.
- D8 and D9:
  - `insert-card` renumbering after the join: survives.
  - Plan-Burler-Fix and Webster reading only the plan directory: survives.
- D13: timing-span and interrupt-policy maps: survives once both rows are added, apart from the reinvoke-then-Probe problem in finding 1.

**Tool calls**
1. Grepped for plan-dir pathspec users, for `Plan-Write` in YAML, and in the settled guard and `generation.go`. My context lacked the landing, commit and settled-guard readers of the plan rows.
2. Read the plan commit pathspec in `wiring.go`, the settled guard's function list and the entry-gate Setting wiring. My context lacked how the plan commit and the writer named in the refusal are formed.
3. Read `PlanReviewSkippable` and the `first_card` users. My context lacked whether rework generations pass through the new rows.
