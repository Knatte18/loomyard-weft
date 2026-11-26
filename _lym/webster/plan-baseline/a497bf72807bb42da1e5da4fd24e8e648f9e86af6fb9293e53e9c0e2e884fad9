# Card 3 — run-watch-engine

**Create:**
- `internal/dobbyengine#runWatchPrefix`
- `internal/dobbyengine#RunWatchDeps`
- `internal/dobbyengine#WatchReading`
- `internal/dobbyengine#OwedWatchLine`
- `internal/dobbyengine#RunWatchState`
- `internal/dobbyengine#daemon.runWatch`
- `internal/dobbyengine#watchStop`
- `internal/dobbyengine#watchLine`
- `internal/dobbyengine#TestRunWatch`

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#State`
- `internal/dobbyengine#daemon`
- `internal/dobbyengine#daemon.tick`
- `internal/dobbyengine#daemon.clearDoneRuns`
- `internal/dobbyengine#TestClearDoneRuns`
- `internal/dobbyengine/doc.go`

**Uses:**
- `internal/dobbyengine#newPassHarness`
- `internal/dobbyengine#passHarness.tick`
- `internal/dobbyengine#passHarness.tickAfter`
- `internal/dobbyengine#passHarness.advance`
- `internal/dobbyengine#passHarness.restartDobby`
- `internal/dobbyengine#passHarness.stateRecord`
- `internal/dobbyengine#loopClock.suspend`
- `internal/dobbyengine#ChildReading`
- `internal/dobbyengine#pairRelative`
- `internal/dobbyengine#oneLineCut`
- `internal/dobbyengine#LandNoticeState`
- `internal/dobbyengine#RunWatchStops`
- `internal/dobbyengine#RunWatchRows`
- `internal/dobbyengine#RunWatchOff`

**Intent:**
The run watch of D1 to D5, in a new `runwatch.go`; the hub wiring comes in card 4.

Seams: `Deps` gains `RunWatch RunWatchDeps`, which holds
`Level func(slug string) (string, error)`, the run's `beam.run_watch` as `LookupRunWatch` answers it, and
`Post func(ctx context.Context, slug, line string) (posted bool, err error)`, which posts one untagged line to the nearest live orch and answers `posted` false with a nil error while no orch strand is recorded.
A nil `Level` or a nil `Post` sends no watch line and keeps no memory, so every existing harness, which sets neither, sees no change (D8).
The child part is read through the existing `Deps.RunNotices.ReadChild`; a nil `ReadChild` leaves the child part absent.

Memory: `State` gains `Watches map[string]RunWatchState` (`json:"watches,omitempty"`), keyed by slug; an older record without the key reads as no memory.
`RunWatchState` holds `Level string` (`json:"level,omitempty"`, the last `run_watch` level read for the run, empty while none read), `Last *WatchReading` (the last reading the watch handled, nil while none), `LandNotice *bool` (the copied land_notice reading, nil while none was copied), `AheadOfLand bool` (the run was not `PastLand` at the last tick that listed it), `Owed []OwedWatchLine`, `GateHolds []string` (card 6 fills it), and `StatusSeen`, `ChildSeen time.Time` (the `RunView.StatusModified` and `RunView.ChildStatusModified` of the last successful child read).
`WatchReading` holds `Run string` (the derived `runfsm.State`), `BattenState`, `BattenRow`, `BattenError`, `ChildRead bool`, `ChildState`, `ChildRow`, `ChildError`, `PairDir`, `Episode int` and `Since time.Time`; it is comparable, and the comparison that decides a change leaves out `Since` and the two error texts.
`OwedWatchLine` holds `Line string`, `OwedAt time.Time`, `Attempts int` and `NextTry time.Time`.
`daemon` gains the flag `watchRecordFailed` (the tick's watch step failed to read or write the state record) and the episode sets `watchLevelReads` and `watchPosts` (`logger.EpisodeSet`).

The step: `tick` calls `d.runWatch(ctx, now)` right after `d.landNotices(ctx, now, d.listedRuns)` inside the scheduling block, so it runs only when the scheduling steps run (D1) and before the done-run clear.
For each run of `d.listedRuns`:
1. The copy: while the run is not `PastLand` and `State.LandNotices[slug]` reads (`LandNoticeState.read`), its `Requested` is copied into `LandNotice` (D3).
   The copy runs whatever the level and before it is read, so a run whose `run_watch` first reads after Land keeps the land_notice reading copied ahead of Land.
   `AheadOfLand` takes `!run.PastLand` on every listed tick.
2. The level: `Level(slug)` is called only while the run is not `PastLand`, or while its entry holds no `Level`.
   Worktree-Teardown removes the task worktree, where the task's own `run_watch` lives, before batten reads done, so a read past Land answers the parent's or the hub's value and succeeds; the level read ahead of Land is the one kept, as the landing notice keeps land_notice.
   A successful read is stored in the entry's `Level`; a failure keeps the stored `Level` and logs one Warn per failure episode through `watchLevelReads` naming the full key `beam.run_watch` and carrying the lookup error, whose refusal names the file, the value and the way forward, with an Info at recovery.
   A run whose entry holds no `Level` skips steps 3 to 5: no reading becomes its memory (D2).
3. The reading: the derived state from `run.Run`, batten's state, row (`Producer`), reason, `HistoryLen` and `LastRepeats` from the `RunView` (card 2).
   The child is read through `ReadChild` only when the entry has no `Last`, or `run.StatusModified` or `run.ChildStatusModified` differ from `StatusSeen` and `ChildSeen`; otherwise the child part of `Last` is kept.
   `StatusSeen` and `ChildSeen` take the run's `StatusModified` and `ChildStatusModified` only after a `ReadChild` that returns no error, with `found` true or false; a failed read leaves both as they were, so the next tick reads the child again.
   A read with `found` false keeps the child part of `Last`, absent when `Last` has none, so a status Worktree-Teardown removes posts no extra line and a run before Run-Shed has no child part.
   A failed read keeps the child part of `Last` and logs once per episode through the existing `noticeFailures` set under `<slug>|watch-child`.
   The episode is batten's `HistoryLen` plus its `LastRepeats` when batten's state is not running, else the child's `HistoryLen` plus its `LastRepeats` when read, else 0, per the Added after Discussion entry "Episode stamp counts folded repeats".
   `Since` is the later of `run.StatusModified` and the child's `Modified`, `now` when both are zero.
4. The judgment: `watchStop` is true when the derived state is paused, halted, cancelling, cancelled or landed, batten's state is not running, or the child part is read and its state is not running (D3).
   A reading equal to `Last` sends nothing.
   Under `RunWatchOff` nothing is owed; under `RunWatchRows` any change owes a line; under `RunWatchStops` a change owes a line only when the new reading is a stop and differs from `Last` in the derived state, batten's state, the child's state, whether the child is read, or the episode (D2).
   A landed reading whose copied `LandNotice` is true owes no line, since the landing notice reports it; a nil copy is never skipped, and a cancelled reading is never skipped (D3).
   A run with no `Last` takes its reading as the change, so a run first seen at a stop owes one line (D4).
   The reading becomes `Last` whether or not a line is owed.
5. The line: `watchLine` renders `[run watch] <slug>: <derived>; batten <state> at <row>; child <state> at <row>`, with `child none` when the child part is absent, then `; <reason>` for the child's reason when set, else batten's, rendered relative to `PairDir` through `pairRelative` and cut to one line through `oneLineCut` at `runNoticeErrorMax`, then `; since <Since in UTC RFC 3339>`.
   An owed line is appended to `Owed` with `OwedAt` now.

Delivery (D5), in the same step, for each entry with owed lines, by sorted slug: lines post oldest first and stop at the first that does not go.
A line whose `NextTry` is after now waits.
A line owed for an hour of awake time, `pauserecord.AwakeSince(OwedAt, now, State.Sleeps)`, is dropped with a Warn before it is tried.
`posted` false with no error leaves the line owed with no attempt counted.
An error counts an attempt, sets `NextTry` to now plus `d.probe`, and logs through `watchPosts`; the third failed attempt drops the line with a Warn.
A posted line is logged to the dobby log as `run watch <slug>: <line>`.

Entry upkeep (D4): the pass step's drop of `State.Runs` never touches `Watches`.
An entry whose slug is not listed this tick, whose `Deps.StatusExists(slug)` answers false and which owes no line is dropped; a failed or nil `StatusExists` keeps it.
An entry whose slug is not listed this tick, whose `AheadOfLand` is set and that `State.LandNotices` holds no memory of has its `LandNotice` copy set to nil, since `rememberRuns` has then dropped the landing memory unposted and the landing notice never remembers a run first seen again past Land; such a run's landed reading is never skipped.
An entry whose run was past Land when last listed keeps its copy, since its landing notice may already have posted and dropped its memory.
`clearDoneRuns` leaves a run whose `Watches` entry owes a line, and, while `RunWatch.Level` and `RunWatch.Post` are both set, a run with no `Watches` entry or whose entry holds no `Level`, so the run waits for a read as an unread landing-notice memory makes it wait.
It drops the run's `Watches` entry in the same state update that drops its counted landing once the clear succeeds.
Every state change goes through `UpdateState`; an unreadable or unwritable record is one Warn and the step ends for the tick.
The step clears `watchRecordFailed` when it starts and sets it on such a failure, and `clearDoneRuns` clears no run on a tick that set it, so a run stays until a later tick records its owed line.

`doc.go` gains a "The run watch" section naming the reading, the three levels, the level kept from ahead of Land and its reason, the stop rule, the land_notice skip, the memory and the delivery rules above.
Its done-run-clear bullet also names a watch line still owed, a run whose watch entry holds no level and a watch step that failed to record, beside the landing notice and the board-tidy count, and so does `clearDoneRuns`'s doc comment.

Tests: `TestRunWatch` is a table over `newPassHarness` ticks with a `RunWatch` whose `Level` answers each row's level (or fails, or fails through a tick) and whose `Post` records lines, can fail a set number of times, and answers not posted while a row says no orch strand is recorded; `RunNotices.ReadChild` answers each tick's child reading, absent or failing as the row says, and counts its calls.
Each row lists per-tick `RunView`s and child readings, and the exact lines posted.
Each tick's `RunView` carries a new `ChildStatusModified` whenever its child reading changes, since the watch re-reads the child only then. Rows:
- a child going from running to blocked at a row posts one line naming the derived state, batten's and the child's state and row and the reason, the exact text pinned once; a later tick with the same reading posts nothing;
- a child blocked, then blocked again at the same row with the same history length and its last entry's repeats one higher, and no running tick between, posts a second line;
- a changed child reading under an unchanged `StatusModified` and `ChildStatusModified` is not read (`ReadChild` is not called) and posts nothing;
- a failing `ReadChild` keeps the child part last read and posts nothing for it; on the next tick, with `StatusModified` and `ChildStatusModified` unchanged and `ReadChild` recovered, `ReadChild` is called and the child's stop posts one line;
- a child reason naming a path under `PairDir` and holding a line break renders the path relative to `PairDir` on one line;
- a child going from running to done under `stops` posts one line, and a derived `cancelling` reading posts one line;
- batten blocked at Land while the child reads done posts one line;
- a run with no child status under `stops` posts nothing while running;
- under `stops`, a row move while running, a resume to running and batten's row moves after the child's done post nothing; under `rows` each posts one line; under `off` no reading posts;
- under `stops`, a child going from blocked to failed posts one line, and a batten pause lifting over a blocked child posts one line;
- a level that has never read posts nothing and keeps no `Last` until it reads, then posts the standing stop;
- a level that reads `rows` and then fails on a tick with a row move still posts that move's line under the kept `rows`;
- an anchor-move record standing (`geom.MoveRecords` naming a file that exists) holds a stop's line, which posts on the first tick after the record is removed;
- an entry of a run no longer listed, whose `StatusExists` answers false and which owes nothing, is dropped; one whose `StatusExists` answers true, and one that owes a line, are kept;
- a rebuilt dobby (`restartDobby`) over the same record posts no line for a reading already told; a fresh record with a run standing at a stop posts one line;
- a landed reading posts one line when the copied land_notice read off or never read, none when it read on; a child status that vanishes at teardown posts no extra line;
- a cancelled reading with land_notice on posts its line;
- land_notice on, copied ahead of Land, then the run unlisted for one tick and next listed landed, posts its landed line;
- land_notice on, the run listed past Land and not yet landed while its landing notice posts, then unlisted for one tick and next listed landed, posts no watch line for the landed reading;
- a level failing through the ticks ahead of Land with land_notice on, then reading after the run is landed, posts no line for the landed reading;
- a level reading `off` ahead of Land, then answering `stops` while the run is listed past Land, across a `restartDobby`, and then landed, posts no line;
- a level reading `stops` ahead of Land, then answering `off` while the run is listed past Land and then landed, posts the landed line once;
- a landed run read on several ticks posts its line once;
- owed lines post oldest first; with `Post` answering not posted on four ticks, more than three, they stay owed with no attempt counted and post once it posts;
- an erroring `Post`, ticked through `tickAfter` at a gap shorter than the probe interval, is not tried again until a probe interval has passed, and is dropped after three failures;
- a line owed an hour of awake time (`advance`) is dropped, and one owed across a `suspend` of an hour is not.

`TestClearDoneRuns` gains a table field that seeds a `Watches` entry for the run.
A new row seeds an entry owing a line, and the run is not cleared.
A second new row seeds the same entry, holding the `Level` `stops`, posts its line through a `Post` that succeeds, and the run is cleared on the next tick.
A third new row seeds an entry with no `Level` and sets a `RunWatch` whose `Level` fails on every tick, and the done run is not cleared; once `Level` reads `stops`, the landed line posts and the run is cleared, by the tick after at the latest.
Every cleared row seeds an entry owing nothing and asserts that the entry is gone after the clear.

**ImpactSummary:** New step and state map inside `dobbyengine`; `clearDoneRuns` gains three holds (an owed watch line, an unread watch level, a watch step that failed to record), and every existing test runs with nil watch seams, so no existing assertion changes.

**Commit:** 3: Watch every run the dobby drives and owe its parent a line per stop
