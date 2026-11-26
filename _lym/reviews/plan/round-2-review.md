---
verdict: BLOCKING
findings:
  - id: F14
    severity: BLOCKING
    class: design
    location: _lym/plan/03-run-watch-engine.md:59
    summary: "Card 3 reads Level for every listed run on every tick, but a run reads landed only after Worktree-Teardown has removed the task worktree, where a task's own run_watch lives, so the lookup succeeds with the parent's or hub's value: a task set to off gets its landed line, one set to stops under an off hub loses it, and the new clear hold reads that same wrong value after a restart."
    fix: "In card 3, make the level a persisted, ahead-of-Land reading, as land_notice is. (1) Memory paragraph: RunWatchState gains Level string (json level,omitempty), the last run_watch level read for the run; drop the in-memory watchLevels map from the daemon fields. (2) Step 2: call Level(slug) only while the run is not PastLand, or while the entry holds no Level; a successful read is stored in Level and a failed one keeps it; a run whose entry holds no Level skips steps 3 to 5. State the reason: Worktree-Teardown removes the task worktree before batten reads done, so a later read answers the parent's value and succeeds. (3) Entry upkeep: the clear hold becomes a run whose entry holds no Level, while RunWatch.Level and RunWatch.Post are both set; the TestClearDoneRuns row with a failing Level seeds its entry with no Level. (4) Tests: add TestRunWatch rows where Level reads off ahead of Land, then answers stops (also across restartDobby) while the run is listed PastLand and then landed, and no line posts; and where it reads stops ahead of Land, then answers off, and the landed line posts once. (5) Have the doc.go run-watch section and the done-run-clear wording say the same."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: resolved
    answer: "Settled by the added entry titled 'Added after Discussion (parent, 2026-11-26): Gate-hold line caps listed processes'; card 6 keeps the cap and cites it."
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: withdrawn
    answer: "The rebuttal holds: tick returns startReady's LoadConfig error (internal/dobbyengine/loop.go:721-722) and passHarness.tick fails the test on any tick error, so the row needs its direct call."
  - id: F13
    status: resolved
---

### [BLOCKING] The watch re-reads a task's run_watch after its worktree is gone

**Location:** `_lym/plan/03-run-watch-engine.md:59`

**Issue:** A task sets `beam.run_watch` through its `run_settings`, which live in the task worktree's local layer.
`hubgeom.TaskSettingChain` leaves out a worktree with no directory and succeeds with the parent's value (`internal/hubgeom/tasksetting.go:20,29-41`).
A run derives as landed only once batten reads done (`internal/runfsm/state.go`), and batten reads done only after Worktree-Teardown has removed the pair.
Step 2 therefore replaces the task's level with an inherited one before the landed reading is judged.
This breaks D2's per-task level, including its acceptance criterion that no reading posts under `off`.
The landing notice avoids this by not reading `land_notice` past Land (`internal/dobbyengine/landnotice.go:75`).
The hold added for an unread level (line 87) keys on the in-memory `watchLevels`, so a restarted dobby reads a done run's level from a chain that no longer holds the task.
The planned rows use a fake `Level` that answers one value throughout, or fails, so none of them reaches this.

## Focus notes

- Clear paths for a run owing a landed line: the owed line, the unread level and the failed watch step each hold the clear, bar F14's wrong level.
  `TestClearDoneRuns` and `TestRunWatch` name no row that reaches the `watchRecordFailed` hold.
- The land_notice copy, through a level first read after Land and through an unlisted tick, against `rememberRuns` and `unreadLandNoticeRuns`: each sequence yields the line D3 asks for.
  Not a defect.
- `StatusSeen` and `ChildSeen` after a failed `ReadChild`: card 3 leaves both unchanged, so the next tick reads the child again, and a row reaches it.
  Not a defect.
- Gate-hold process list: the cap is licensed by the added entry.
  Not a defect.

## Rejected

These are real but are not raised, since an incremental round raises a new finding only at BLOCKING on a changed hunk, and none of these is BLOCKING.

- The missing `watchRecordFailed` test row in card 3, noted above; it could be reached by making the state directory read-only through `permkit.Chmod` and calling `runWatch` and then `clearDoneRuns` directly.
- Card 3's `WatchReading` keeps no child history count or child modification time.
  A tick that keeps the child part, such as one after the child status vanishes at teardown, has nothing to compute the child's episode and `Since` from.
- Card 6 groups slots by PID, but a report step of the same `lym gate test` process holds a slot with no tier and no marker, which can be the earliest (`internal/gatecli/redundancy.go`, `holdSlot`).
  The card does not say which slot gives the line its tier, site and marker.
- Card 6 runs the gate-hold check inside the scheduling block, so an anchor move, adopt hold or owed catch-up delays it, while D6 says each probe tick and only D1 states that delay.
