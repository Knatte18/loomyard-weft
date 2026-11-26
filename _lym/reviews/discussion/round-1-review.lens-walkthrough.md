```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end against the code.

**Decisions walked**
- D1, child blocks at row X: the derived state turns halted, the child's state turns blocked and the stamp is the child's history length. A block appends history (`internal/shedengine/run.go:418-464`). Purpose met.
- D1, cancel with `--remove` (or a cancel whose finish finds no record): the records are removed before any tick reads `cancelled`. Purpose not met (F3).
- D2, `stops` with batten's row moving after the child's done: the child's state stays done, batten's stays running, the derived state goes running → landing (not a stop) and the stamp doesn't change. Purpose met.
- D3, landed run with `land_notice: on`: purpose not met (F2).
- D4, landed run (batten done, no pass in flight): purpose not met (F1).
- D5, no orch strand: `notifyPrime` returns `(false, nil)` when no strand is recorded (`internal/orchcli/notify.go:263-266`), so "waits" can be told apart from "failed". The existing `Deps.Notice` folds that case in through `requirePosted` (`internal/beamcli/dobbydeps.go:247-248`), so a new seam is needed. That is Plan's call. Purpose met.
- D6, tier holder under the task worktree: `Pool.Holders` gives `Worktree`, `PID`, `Started`, `Tier` and `Slot` (`internal/gateslot/pool.go:73-86`). The marker field is new and the rest of the chain is consistent. The dedup key lives in the run's entry, and the run is not done while it holds a gate, so it survives. Purpose met.
- D7, D8: documentation only and no mechanism. Not applicable.

**Findings**

F1. BLOCKING · design · D4 (decision-record.md:78-86)
D4 keeps the watch memory and owed lines in the run's state entry, but every tick deletes that entry for a done run before the watch step runs. The result is a landed line reposted every tick.
- `drivePasses` drops done runs that have no pass in flight from `runs` (`internal/dobbyengine/passes.go:184-188`).
- `recordOwed(runs)` then deletes each `s.Runs` slug not in that list (`internal/dobbyengine/passes.go:333-336`). It runs at `tick` line 710, ahead of the steps at 717-729 (`internal/dobbyengine/loop.go:710-729`).
- Once batten reads done, every tick erases the watch memory and the owed queue, so the watch sees no memory. Under D4's no-memory rule it posts the landed line again, every tick, until the clear.
- The retry and attempt counts reset every tick, so the three-failure drop never happens.
- "The done-run clear keeps a run's entry while it owes a watch line" relies on an entry `recordOwed` has already deleted.

F2. MEDIUM · design · D3 landed skip (decision-record.md:70, 76; AC line 167)
The skip never fires, so a run with `land_notice: on` gets both the landing notice and a landed watch line.
- `PastLand` is already true at the Worktree-Teardown row while batten still reads running (`internal/battenrecipe/land.go:27-28`).
- On that tick `rememberRuns` decides the notice. `settleLandNotices` posts it and `dropLandNotice` deletes the memory (`internal/dobbyengine/landnotice.go:142-149, 171-177`). With the setting off, the memory is deleted at once (lines 145-147).
- The derived `landed` state needs batten's state to be done (`internal/runfsm/state.go:71-73`), which only happens on a later tick.
- So when the watch first reads `landed`, the landing-notice memory is already gone in either tick order. The record's claim that the two "never disagree", and the acceptance criterion "none when it reads requested", both fail.
- The skip works only while the landing notice's own post keeps failing.

F3. MEDIUM · design · Goal "cancelled" / D1, D3 (decision-record.md:9, 33-46, 66)
A cancel that removes its records takes the run out of the dobby's list before any tick reads it as cancelled, so the Goal's "cancelled" stop gets no line.
- `clearRunPhase` with `remove` calls `removePairAndBranch` and then `removeRecords`. A kept cancel whose `FinishCancel` finds no record also removes the records (`internal/battencli/cancel.go:544-565`).
- `dobbyParentRuns` lists only slugs whose batten status file exists (`internal/battencli/dobbyreads.go:104-111`).
- At most, a tick that happens to fall inside the cancel window reads `cancelling`. The record doesn't say that removed cancels go unreported or how to catch them.

**Tool calls**
1. Read `clearRunPhase` in `internal/battencli/cancel.go` and grepped the child-reader wiring: needed to know whether a cancel removes the records the run listing reads.
2. Read `recordOwed` in `internal/dobbyengine/passes.go` and grepped the shed history appends: needed to see what happens to `s.Runs` for done runs, and whether a block grows the history.
3. Grepped the `Notice` wiring in `internal/beamcli` and read `notifyPrime` in `internal/orchcli/notify.go`: needed to see how "no orch strand" comes back from the post path.
4. Read lines 240-275 of `internal/beamcli/dobbydeps.go`: needed the `Notice` seam's exact body (`requirePosted`).
