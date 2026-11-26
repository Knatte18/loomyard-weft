```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code that already writes or reads each state the record adds.

**Paths traced, per decision**

- **D1/D4: watch memory in the run's state entry vs `recordOwed`.** BREAKS. `drivePasses` hands `recordOwed` only the runs left after the done filter, so a done run's `s.Runs[slug]` is deleted on every tick before any later step runs (`internal/dobbyengine/passes.go:184-189`, `:333-336`; the doc at `:171` says "its state entry is dropped").
- **D1: which run list the watch reads.** BREAKS unless the plan says. `d.runs` excludes a done run that has no pass in flight (`internal/dobbyengine/passes.go:184-188`), and `runNotices` iterates `d.runs` (`internal/dobbyengine/runnotice.go:182`). Only `d.listedRuns` (`passes.go:181`) still holds a run whose batten reads done, which is how a landed run derives (`internal/runfsm/state.go:71-76`). The record does not say which list the watch reads.
- **D3: landed/cancelled skip vs `rememberRuns`, `settleLandNotices` and `dropLandNotice`.** BREAKS.
- **D4: done-run clear vs `clearDoneRuns`.** Survives in shape. It already holds a run on `current.LandNotices` (`internal/dobbyengine/donerun.go:45`), and a similar test on owed lines can be added. Whether it works depends on where the memory lives (finding 1).
- **D4: restart and re-exec.** Survives. The state record is re-read on every tick, and a re-exec keeps it (`internal/dobbyengine/state.go:86-87`).
- **D1: tick gating.** Survives as the record words it ("every tick that runs its steps"). The run steps and the clear both run only while `scheduling` holds (`internal/dobbyengine/loop.go:708-729`), so a watch line waits out an anchor move, an adopt hold or a catch-up. The record does not say so.
- **D2: per-task check.** Survives. `LoadConfig` and `CheckLocalValues` loop over a fixed key list (`internal/dobbyengine/config.go:130`, `:146`), and `run_watch` joins it. `checkTaskSetting` (`config.go:162-169`) is keyed per value list, so it needs a `run_watch` branch, which D2 implies.
- **D5: delivery seam vs the existing `Notice`/`Post` wiring.** Partly breaks. Every existing dobby seam turns "no orch strand" into an error through `requirePosted` (`internal/beamcli/dobbydeps.go:89-97`, `:247-249`). D5 needs the two told apart and names `orchcli.NotifyRunFrom`, which returns `posted` (`internal/orchcli/notify.go:73`). The record never says the watch's seam must carry `posted` instead of going through `requirePosted`.
- **D6: slot holder record writer.** Survives. `Holder` is written at `Pool.Acquire` and `AcquireSlots` (`internal/gateslot/pool.go:130`, `:179`). The tier marker already exists in the tier run's environment (`internal/gatecli/processcap.go:144-150`), so it can be passed into the `Holder` at acquire. The record is YAML decoded non-strictly (`pool.go:360-361`), so an older record without the field reads as no marker.
- **D6: gate config source.** Survives. The gate spec is `HubWide: true` and `tiers` is one executable key (`internal/gateslot/config.go:192-208`), so there is one hub-level `timeout_min` per tier and no per-worktree choice.
- **D6: dedup key in the run's state entry vs `recordOwed`.** Survives for a running run. A done run's key is wiped (finding 1), but a done run holds no gate.

**Findings**

1. **BLOCKING, design, `decision-record.md` D4 (also D1).** The watch memory and owed lines kept in "each run's entry in the dobby's state record" are deleted by `recordOwed` on every tick once batten reads done. A landed or cancelled run then loses its owed line, and comes back with no memory. Under D4's no-memory rule it posts its stop line again on every tick until the clear removes it, or never posts it.
   - Evidence: `internal/dobbyengine/passes.go:171`, `:184-189`, `:333-336`; `internal/dobbyengine/donerun.go:38-45`.
   - Fix: keep the watch memory in its own map keyed by slug, as `LandNotices` is (`internal/dobbyengine/state.go:46`), dropped only by the watch or the clear. Have D1 read `d.listedRuns`.
2. **BLOCKING, design, `decision-record.md` D3 (skip rule).** The skip "landing-notice memory … holds it as requested" never fires.
   - `PastLand` turns true at the Worktree-Teardown row (`internal/battenrecipe/land.go:27-29`). On that tick `rememberRuns` decides the line, and `settleLandNotices` posts it and deletes the memory (`internal/dobbyengine/landnotice.go:142-149`, `:171-177`). A run with the setting off has its memory deleted at once (`:145-147`).
   - Derived `landed` needs batten's state to be done (`internal/runfsm/state.go:71-76`), a later tick. By then the memory is empty in either step order.
   - A run with `land_notice` on therefore gets both the landing notice and a watch landed line, which contradicts D3's rationale and its acceptance criterion "none when it reads requested".
   - Fix: decide the skip from a record that outlives the landing notice, such as a flag the watch copies while the run is ahead of Land, or `LandNoticeState.Requested` read before PastLand.
3. **LOW, consistency, `decision-record.md` D5.** The existing dobby notice seams turn "no orch strand" into an error (`requirePosted`), so a plan reusing `Deps.Notice` or `RunNotices.Post` cannot tell the two apart. D5's "no strand waits and counts no attempt" holds only if the new seam passes on the `posted` result of `NotifyRunFrom`. The record does not say so.
   - Evidence: `internal/beamcli/dobbydeps.go:89-97`, `:247-249`; `internal/orchcli/notify.go:73`.
4. **LOW, design, `decision-record.md` D1.** The run steps run only while `scheduling` holds, and D1 says nothing about it.
   - During an anchor move, an adopt hold or an owed catch-up, the watch posts nothing (`internal/dobbyengine/loop.go:708-719`). A catch-up owed after a machine sleep can delay stop lines.
   - Fix: state the hold as a bound, or run the watch read and post outside the `scheduling` gate. It sends only mail and changes no run.

**Tool calls**

1. `recordOwed` in `internal/dobbyengine/passes.go`: the manifest lacked how the per-run state entry is pruned.
2. A grep of the `Notice`/`Post` seams in `internal/beamcli`: the manifest lacked the hub wiring of the dobby notice seams.
3. A grep of `Acquire` in `internal/gateslot/pool.go` and of the marker in `internal/gatecli/processcap.go`: the manifest lacked where the holder record and the tier marker are formed.
4. `requirePosted` and the `Notice` wiring in `internal/beamcli/dobbydeps.go`, and `postDobbyNotice` in `internal/beamcli/dobbytrace.go`: needed to see how an unrouted post maps to an error.
