```yaml
lens: existing-paths
outcome: reported
```

The existing-paths lens traced what each decision runs into in the current code. Two decisions break on an existing path and one is unclear. The rest survive.

**Paths traced, per decision**
- D1, `segmentWriter` (`internal/loomcli/settledguard.go:97`): it picks the row whose `OnDone` is the Bouncer, which is `Plan-Write`. D1 replaces that with a read of the run history and says so. **Survives.**
- D1, the bounce-budget code (`episodeStuckCount`, `segmentEnders`, `internal/shedengine/run.go:631-693`): `Plan-Write` has no recipe segment (`contracts/recipes/tapestry-recipe.yaml:226-228`), so the new rows' "segment plan" is a model segment and changes no budget count. **Survives.**
- D1, routes that name `Plan-Write` in the recipe: there are none, only `Drift-Baseline`'s `on_done` (`tapestry-recipe.yaml:222`), and D1 changes that edge. **Survives.**
- D13, Go code that names the `Plan-Write` row: the interrupt policy (`internal/loomshed/interruptpolicy.go:36`) and the timing phase (`internal/reportengine/timingspans.go:75`). D13 adds both new rows to each. **Survives.**
- D4 and D6, order of the fresh-spawn steps: `prepareFreshSpawn` runs before `archiveStaleOutputs(table.Outputs())` (`internal/shedadapters/multillm.go:115-123`). The part directories and `join.md` are moved out first, so the producer's own archive finds nothing to move. **Survives.**
- D6, the table source runs before the probe and reads the parts file, which the preparation keeps (`multillm.go:94-101`). **Survives.**
- D6, the commit decorator commits on any non-empty pointer, so the new writer Stuck commits too (`internal/loomshed/planwrite.go:73-93`). **Survives.**
- D5, resume of a writer the probe did not find: the outputs-plus-Accept rule replaces today's step end for a lost required seat (`internal/seatengine/resume.go:117-124`). **Survives**, except for the case below.
- D5, `Probe` removing a done writer whose pane is kept (`resume.go:22-23, 39-46`), against a fix round in progress: **breaks** (Finding 1).
- D5, `outstanding` skipping ended runs (`internal/seatengine/work.go:47-49`): D5 changes this and says so. **Survives** on the fresh path; on resume it is undone by the same `Probe` removal.
- D6, `watch` ending the step on a writer that timed out (`internal/seatengine/engine.go:127-137`), read through D6's mapping: **breaks** (Finding 2).
- D6, the per-writer gate-findings file and `clearGateFindings`, which only clears the chair's file (`multillm.go:168-170`): **unclear** (Finding 3).

**Findings**

1. MEDIUM, design. D5 resume rule with D9/D11 fix rounds and D13 reinvoke.
   - Problem: on a reinvoke, `Probe` stops any writer whose run record is done but whose pane is kept, even one busy on a fix the chair asked for.
   - What follows: `Resume` then judges the part on whatever is on disk. If the fix was cut off mid-edit, Accept fails and the run goes Stuck. If Accept passes, the chair is told the part stands and gets no word that its fix request was lost.
   - Gap: the record extends the hold deferral to a done writer that is mid-turn, but never says how the probe treats that writer at resume.
   - Evidence: `internal/seatengine/resume.go:22-23` (the probe removes a done seat kept by its pane), `resume.go:39-46`; decision record lines 118, 125.

2. MEDIUM, design. D6 Stuck mapping.
   - Problem: a required writer that times out ends the step (`watch`, `internal/seatengine/engine.go:127-137`: anything other than an accepted done calls `ender.end`).
   - Gap: D6 maps only two cases. A writer that ended done with a failed gate or Accept, or one not found at resume, becomes Stuck. A writer that failed to start or died stays an error. Timeout fits neither.
   - What follows: under today's `mapOutcome` the stopped chair produces an error that names the chair, with nothing committed, which is the result D6 exists to prevent. Acceptance criterion D6 does not cover a timed-out writer.
   - Evidence: `engine.go:133-137`, `internal/shedadapters/multillm.go:173-183`; decision record lines 142-145.

3. LOW, design. D6 per-writer gate findings.
   - Problem: each writer's findings file (key `Parts-Plan-Write-&lt;part&gt;`) is written to run state, but the only existing clear runs on the chair's own findings path when the chair's gate passes.
   - What follows: a writer that failed in one attempt and passed in a later rerun leaves its old findings file behind.
   - Unverified: I ran out of tool calls before finding whatever reads that file, so I can't say whether a stale file is ever read.
   - Evidence: `internal/shedadapters/multillm.go:168-170` (`clearGateFindings(p.gateFindingsPath)` only); decision record line 137.

**Tool calls**
1. `segmentWriter` and the code that reads segments: the record changes how the plan writer is named, and none of that code was in my context.
2. Segment uses in `run.go`, `progress.go`, `checklist.go`, `wiring.go`, `bouncer.go` and `entries_settings.go`: to check whether the new rows' segment touches budget or checklist code.
3. `run.go` bounce-budget code and an attempt to read the tapestry recipe: to check whether a skip or a Stuck changes the budget count.
4. A search for the tapestry recipe file: the previous call did not find it.
5. The tapestry recipe's `Plan-Write` row and the gate-findings path helpers: to learn `Plan-Write`'s segment and who reads gate-findings files.
6. Routes and Go code that name `Plan-Write`: to find paths that would bypass the switch.
