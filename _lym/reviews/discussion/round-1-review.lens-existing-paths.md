```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. For each state the record introduces or changes, I traced the code that already writes or reads it.

**Paths traced, per decision**

- D2, a task carrying the `sketch` label: `boardengine.IsSketch` (`sketch.go:35`) reads kind first, so such a task is no Sketch. The other readers, `boardcli/text.go:61` and the Sketches heading in `render.go:149-152`, see a task. **Survives.**
- D2, promote with a recipe: `stampDates` treats a promote as no revision (`dates.go:43-46`). `checkTaskCreation` refuses only a write from a run agent's strand (`taskguard.go:21-23`), so the operator's promote passes. **Survives.**
- D9, marking the Sketch done in the run's own board write: `checkRunLocks` ignores Status (`runlock.go:185-217`), so a done status alone passes. In the same write, new task entries are judged by `checkTaskCreation` with the board's strand name. That name is set only by `boardcli` from the environment (`boardcli/cli.go:159`, `board.go:155`), so Go code opening the board directly carries none. **Survives.**
- D9 against later status writes: `Board.SetRunStatus` skips a done entry (`board.go:203`). Every run-status writer goes through it: loom (`loomcli/wiring.go:202, 237, 273`), pawl (`pawlwiring.go:58`) and batten (`battencli/commitstatus.go:143`). `MakeSketch` also skips a done entry (`sketch.go:77`), so the end-as-Sketch finish writes nothing even before D10's row rule. **Survives.**
- D9 against Land's prune: `Store.Prune` removes done entries, except those carrying `notify_on_land` (`store.go:793`). **Survives**, with that exception.
- D8, the `revised` stamp: `stampDates` stamps `revised` only on a scope change, and a status change is no revision (`dates.go:19-21, 47-49`). A weighed entry that is only running moves through status writes without failing the re-check. The same rule leaves profile, `run_settings` and the `discussed` flag unstamped (`runlock.go:191`), so a change to those passes unseen. **Survives** as the record intends.
- D3, the stored record: `Board.DecisionRecord` requires the `discussed` flag and the file (`decision.go:64-80`). Its readers are `seedDiscussedRecord` (`loomcli/discussionrecord.go:54`) and `lym board get --record` (`boardcli/cli.go:764-775`). **Survives**, provided each filed entry's flag and file are written together.
- D7 and D8, the approval record across a Stuck return to Raddle-Write: **breaks** (F1).
- D10, the raddle store beside `board.json`: no board path sweeps unknown files. The only removals are named paths (`board.go:476`, `render.go:59`, `decision.go:97, 114`, `replay.go:592`). It survives prune. How the store reaches Bolt's origin and other machines: **not stated** (F2).
- D12, `SessionUsage` gaining split counts: `roundUsage` and `burlerengine` read `Fresh` and `CacheRead` (`claudeengine/usage.go:295`, `burlerengine/engine.go:87`). "Gains" is additive. **Survives** if `Fresh` keeps its meaning.
- D12, the usage files: they live in the tracked run tree and are committed by the run's status commits. Reports read finished runs from archived trees (`reportengine/archivedrecords.go`). **Survives.**

**Findings**

F1: MEDIUM, design, D7 with D8 (a Stuck return to Raddle-Write in interactive mode).
- **Summary:** The approval stays valid across a return to Raddle-Write. The record never says it is cleared on that return, so a proposal left unchanged can be filed with no new approval.
- **How it happens:** D8 says the revised proposal needs a new approval. But Raddle-File checks only that the file's hash matches the recorded approval (D7: "Raddle-File files only a proposal whose hash matches the recorded approval"). Suppose a weighed entry moved and the agent's new session leaves the file unchanged, or restores it. It reruns `raddle-propose`, which moves the re-check's baseline forward (D8: "measures from the latest passing check"). The old approval's hash still matches, so Raddle-File files with no operator approval of the moved board.
- **Fix:** D7 or D8 should state that a Stuck return clears the approval record, or that the approval binds to the re-check baseline it was given against.
- **Evidence:** `decision-record.md:86-90` and `decision-record.md:95, 102`.

F2: LOW, design, D10 (the raddle store) with D9 step 1.
- **Summary:** The raddle store is written to the Bolt board directory outside the board's own write path. The record does not say how it is committed or pushed.
- **Why it matters:** The board's cross-machine replay handles entries and their decision files (`replay.go:190`, `replay.go:592`). D10 does not say whether the store goes through `boardengine`'s write and commit, or through some other committed path. A Raddle-File crash between step 1 and step 3 is covered by D9's "skipped when already there", but a push of the store is not.
- **Uncertainty:** I did not trace the board's commit path. The evidence is that the replay code deals only in entry fields and decision files.
- **Evidence:** `decision-record.md:110, 127`; `internal/boardengine/replay.go:190, 592`.

F3: LOW, consistency, D9 Bound ("batten's Land prunes it").
- **Summary:** Land does not prune a done entry that carries `notify_on_land`, so the claim does not hold for every Sketch entry.
- **Why it matters:** `Store.Prune` keeps such entries (`store.go:793`). After promotion, the raddle task can carry `notify_on_land`, and `sketchOf` clears the field only when an entry becomes a Sketch, not when it is promoted. In that case the Sketch entry and its `decision-<slug>.md` stay past Land. Only the Bound's wording is wrong; D10's store makes the outcome harmless.
- **Evidence:** `decision-record.md:123`; `internal/boardengine/store.go:793`; `internal/boardengine/sketch.go:58`.

**Tool calls**

1. grep of `revised` stamping and `Store.SetStatus`: the manifest lacked which writes bump `revised`, which D8 compares.
2. `dates.go` and the readers of `IsSketch`/`SketchType`: needed for D8's stamp rule and the D2 label readers.
3. grep of `.Discussed` readers and sketch validation: needed to find the paths that read the flag D3 and D9 set.
4. `decision.go` `DecisionRecord`, its callers, and the `--record` read in `boardcli`: needed for the D3 and D9 record readers.
5. Header of `reportengine/archivedrecords.go` and the functions in `source.go`: needed for how reports reach a finished run's tracked records (D12, D13).
6. grep of removals in the board directory and of the critical section's commit: needed for whether any path sweeps the D10 store, and for its commit route (inconclusive).
