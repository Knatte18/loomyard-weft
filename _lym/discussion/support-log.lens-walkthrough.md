```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision of the dobby-run-watch decision record end to end on one concrete case and checked the end state against the decision's purpose.

**Per decision**
- **D1** (case: the child goes from running to blocked at Webster): met, with one gap. The child status files are in `dobbyWatchDirs`, so the change wakes a tick within a second and the reading changes. A stop that is resolved and stops again within the one-second coalesce, or between ticks while `scheduling` is false, is never seen (LOW, below).
- **D2** (case: `rows` with a row change while running): met. The out-of-set wording is inconsistent (LOW, below).
- **D3** (case: landed with `land_notice` on, then off): partly met. After teardown the child reading goes from done to absent, and that produces an extra stop line (MEDIUM, below).
- **D4** (case: a dobby re-exec while a run is blocked): met for re-exec. It is broken by the done-run clear (MEDIUM, below).
- **D5** (case: the first post fails and the second tick succeeds): met, unless the clear removes the run between attempts (same finding as D4).
- **D6** (case: a tier3 hold of 35 min × 1.5 by a hung testhost, as in #216): partly met.
  - The holder's `Worktree` is the task worktree path (`gatecli/run.go` `Holder{Worktree: location.WorktreePath()}`), so matching works.
  - The pair-dir input and the orphaned-testhost tree both miss (two MEDIUM findings, below).
- **D7**: review-only; no mechanism to walk.
- **D8** (case: a blocked child at Webster): met. batten's notice is unchanged and the watch line comes first.

**Findings**

1. **MEDIUM — D4/D5: the done-run clear can drop an owed watch line.**
   - `clearDoneRuns` holds a done run only while `LandNotices` remembers it, its pair stands, or a pass or lock is live.
   - D4 keeps the watch memory in the run's state entry, which "goes with the run's entry when the dobby clears the run". D5 retries a failed post on later ticks.
   - So a landed line whose first post failed (no orch strand recorded) is lost once the pair is gone and the clear runs before the retry. That defeats D5's "the landed line included".
   - Evidence: the conditions in `internal/dobbyengine/donerun.go` `clearDoneRuns`.
   - Fix: the clear waits while a watch line is owed for the run, as it waits for the land notice. Pin that the watch step runs before `clearDoneRuns` in the tick.

2. **MEDIUM — D3/D1: the child reading going from done to absent after teardown yields a second, spurious stop line.**
   - batten reads done only after Worktree-Teardown, so the pair's child status file is gone before `landed` derives.
   - The reading's child part changes from "done at Finalize" to absent. "Child state not running" makes that a stop, so a line posts even with `land_notice` on, where D3 means to send no landed line.
   - Fix: define how an absent child reads in the reading (for example, carry the last child part, or ignore an absent child once the derived state is landing, landed or cancelled).

3. **MEDIUM — D6: the step's input is missing.**
   - Matching a holder to a run needs each driven run's task worktree directory on every probe tick.
   - D1 reads the child, the only source named for the pair dir (`ChildReading.PairDir`), only when a status file changed. On a quiet probe tick during a 21-hour hang there is no fresh pair dir unless it is remembered.
   - Fix: state the pair-dir source for the gate check (for example, the pair location the dobby already resolves, as `dobbyWatchDirs` does through `pairLocation`), independent of D1's changed-only read.

4. **MEDIUM — D6: the purpose is missed in the entry's own fault.**
   - `proc.TreeSample(holder PID)` lists only descendants still parented under the holder.
   - #216's "likely area" is a timeout that killed `sh` and left an orphaned testhost holding pipes. An orphan is reparented to init or a subreaper, so the notice would name `lym gate` and miss the hung testhost the entry asks to name.
   - Fix: also name the processes carrying the gate run's env marker (`proc.PIDsWithEnv`, `envmarker_linux.go`), or state the bound that orphans are not named.

5. **LOW — D2: the wording on an out-of-set value contradicts itself.**
   - It says the value "refuses the dobby's lookup … and the run reads as `stops`". A refusal and a fallback are different outcomes; pick one, as the acceptance bullet says "reads as `stops` and is logged".

6. **LOW — D1/D4: a stop the arbiter settles and repeats within one coalesced tick is never told.**
   - Case: blocked at X, the arbiter resumes the run, and it blocks at X again within about a second, or while `scheduling` is false because a catch-up or adopt hold stands. The remembered reading equals the new one, so no line goes.
   - Fix: add the child status `since` or the history length to the reading key, as the run notices stamp their episodes.

**Tool calls**
- Read `dobbyengine/loop.go`'s tick, `beamcli/dobbywatch.go` and the probe wiring, to learn the tick order and whether child status files wake the dobby. My context lacked the tick body and the watch dirs.
- Read `dobbyengine/donerun.go`, to learn when a landed run's entry is cleared relative to owed lines. My context lacked the clear's conditions.
- Grepped the gate holders' `Worktree` field at the `gatecli` acquire sites, to check that a holder maps to the task worktree. My context lacked the acquire sites.
