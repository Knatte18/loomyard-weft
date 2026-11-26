```yaml
lens: guard-reach
outcome: reported
```

Guard-reach list (per guard or added row: reachable or not):

**Card 1**
- `checkTaskSetting` refusal for `run_watch` outside its three values, at a worktree layer, at `LoadConfig`, and at `CheckLocalValues` through `configreg`: reachable.
- `LoadConfig` refusal of a negative `gate_hold_factor` or a `gate_hold_min` below 1: reachable through a worktree layer and through `lym config beam --set`, since `configreg` registers `checkLoad(dobbyengine.LoadConfig)`.

**Card 3**
- Step 1 skip of a run whose level never read: reachable. The same "not judged" state recurs unguarded in `clearDoneRuns` (finding 2).
- Child re-read gate on `StatusSeen`/`ChildSeen`: reachable. What the gate does after a failed `ReadChild` is unstated, and the planned row does not reach it (finding 3).
- `found == false` keeping the child part: reachable. Before Run-Shed and after teardown, `modTimeOf` answers a zero time for a missing file (`internal/battencli/dobbyreads.go:304-308`), and that change re-opens the gate.
- Landed skip on a copied `LandNotice` of true: reachable. It also fires in a state where the landing notice has already forgotten the run and sends nothing (finding 1).
- A nil copy is never skipped, and a cancelled reading is never skipped: reachable.
- The `posted` false wait: reachable. `NotifyRunFrom` answers false while no strand is found (`internal/orchcli/notify.go:73-74`).
- Retry after `NextTry`, the drop after three failures, and the drop after one hour awake: reachable. `runWatch` runs on every scheduling tick, so ticks closer together than the probe interval occur.
- Entry drop for an unlisted slug with no status that owes nothing: reachable, after a `--remove` cancel removes batten's status.
- An entry kept while `StatusExists` fails: reachable.
- `clearDoneRuns` hold while a watch line is owed: reachable.
- A state-record write failure ending the step: reachable. Its interaction with the clear is finding 5.
- Row for a standing anchor-move record: reachable through `tick`'s `standingMove`.

**Card 5**
- A marker recorded by `acquireSlot` with `os.Getpid()`, matching the entry `runTest` writes: reachable in one process (`internal/gatecli/run.go:172`, `:705`, `:822`). A nested run inside an inherited slot writes a different marker into the environment of the processes it starts (finding 4).

**Card 6**
- Nil `Holders` or `Post`, a non-probe tick, factor 0, the sibling-prefix worktree, no tier falling back to `GateHoldMin`, an empty marker skipping `MarkedPIDs`, a failing `Holders`, `TaskWorktree`, `Tree` or `MarkedPIDs`, and a failing `Post`: reachable.
- The failing-`LoadConfig` row: the guard is reachable through `tick`. The row's stated reason for bypassing `tick` is false (finding 6).

**Card 7**
- `dobbyGateHolders`' wrapped gate-config load failure: reachable, since `gateslot.LoadConfig` is strict.
- The empty-tier and undeclared-tier rows giving a zero `TierTimeout`: reachable.

**Findings**

1. MEDIUM, decision. Location: `_lym/plan/03-run-watch-engine.md`, step 2 and the landed-skip rule.
   Summary: the watch keeps its copied `LandNotice` of true after the landing notice has dropped the run's memory, so a landed run gets neither line.
   - Path: a run ahead of Land with `land_notice` on is copied as true. On one tick `DobbyRuns` leaves it out, because its records are unreadable or its code tip does not settle.
   - On that tick `rememberRuns` drops the land-notice memory of the unlisted run, since it owes nothing.
   - The run is then first seen past Land, and such a run "is never remembered", so no landing notice goes.
   - The watch entry survives, because `StatusExists` is true and a run "left out of one tick's list keeps its entry". Its copy stays true, so the watch skips the landed line too.
   - The fix: drop or clear the copy whenever the landing notice holds no memory of a run that is still ahead of Land. Equivalently, skip only while `State.LandNotices[slug]` existed on the run's last tick ahead of Land.
   - Evidence: `internal/dobbyengine/landnotice.go:117-118,159-163`; `internal/battencli/dobbyreads.go` (the "left out of the call" doc on `DobbyRuns`).

2. MEDIUM, design. Location: `_lym/plan/03-run-watch-engine.md`, step 1 and Entry upkeep.
   Summary: the "a run whose level never read is not judged" guard has no counterpart in `clearDoneRuns`.
   - A done run whose `run_watch` never read owes no line, so the clear removes it and its watch entry.
   - Its landed line is then never judged, unlike the landing notice: there, an unread past-Land run stays remembered and holds the clear.
   - The fix: hold the clear while the run's level has never read, or state the drop as accepted.
   - Evidence: `internal/dobbyengine/donerun.go:42-44`; `internal/dobbyengine/landnotice.go:100-108`.

3. MEDIUM, design. Location: `_lym/plan/03-run-watch-engine.md`, step 3 child-read gate and the test row "a failing ReadChild keeps the child part last read".
   Summary: the card does not say that `StatusSeen`/`ChildSeen` advance only on a successful read, and no row reaches the tick after a failure.
   - If the seen times move on a failed read, the child change behind the failure is missed until the next status write. A child halted at a stop writes nothing more, so the stop line is never sent.
   - The fix: state that a failed read leaves both seen times unchanged. Extend the row so the next tick, with unchanged mod times and `ReadChild` recovered, reads the child and posts the stop.
   - Evidence: the card's Memory paragraph ("of the last child read"); `internal/dobbyengine/passes.go:349-350` shows the analogous `childSeen` set unconditionally.

4. LOW, design. Location: `_lym/plan/05-gate-holder-marker.md`, Intent.
   Summary: the recorded marker is not the entry carried by the processes of a nested gate run inside an inherited slot.
   - A nested run takes no slot (`e.inherited`) but rewrites `LYM_GATE_TEST_PID` to its own PID for everything it starts.
   - An orphan of a nested tier therefore carries a marker that `PIDsWithEnv(holder.Marker)` never matches, which is the #216 shape D6 targets.
   - The fix: state the bound in the Intent ("an orphan of a nested run is not named"), or have the nested run keep the enclosing marker.
   - Evidence: `internal/gatecli/run.go:154-156,172,696`.

5. LOW, design. Location: `_lym/plan/03-run-watch-engine.md`, "an unreadable or unwritable record is one Warn and the step ends for the tick".
   Summary: when the state record reads but does not write, the owed landed line is never recorded.
   - `clearDoneRuns` then reads the record, sees no owed line, and clears the run, so the landed line is lost.
   - The fix: name this bound, or skip the clear on a tick whose watch step failed to record.
   - Evidence: `internal/dobbyengine/donerun.go:30-34`; `internal/dobbyengine/loop.go:715-717` (the clear runs after the watch regardless).

6. LOW, consistency. Location: `_lym/plan/06-gate-hold-engine.md`, test row "a failing LoadConfig posts nothing…".
   Summary: the row calls `d.gateHolds` directly "since startReady fails the tick", but `gateHolds` runs before `startReady` in the scheduling block.
   - A failed `LoadConfig` only marks the tick failed, and the loop goes on, so `tick` reaches the guard.
   - The fix: drive the row through `tick` (probe tick) like the others.
   - Evidence: `internal/dobbyengine/loop.go:709-716,583-584`; `internal/dobbyengine/starts.go:93-95`.

Tool calls:
- `internal/gateslot/config.go`, `internal/gateslot/tiers.go`, `startReady`'s config load and the loop's tick-error handling: to check the tier timeout field and whether a failed beam config load stops the loop before `gateHolds`. The manifest lacked them.
- `gateslot` `TimeoutMin` validation and `battencli` `modTimeOf`: to check how a missing child status file reads in `RunView`. The manifest lacked them.
