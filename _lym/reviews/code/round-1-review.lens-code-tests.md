```yaml
lens: code-tests
outcome: reported
```

Task: lens `code-tests`, finding for each behaviour the diff adds or changes the test row that would fail if it broke.

## Behaviour → pinning test

**Settings and run view (cards 1, 2)**

| Behaviour | Pinned by |
|---|---|
| `run_watch` values and refusal, `gate_hold_*` ranges and defaults | `TestLoadConfig`, `defaultsWith` |
| Per-task chain lookup and refusal | `TestLookupTaskSettings`, `…_RefusesAValueTheKeyDoesNotTake` |
| `run_settings` check, help line | `TestPrepareRunSettings`, `TestHelpSchema_LeafCommands` |
| `RunView` history, repeats and reason; child repeats | `TestDobbyRuns_Reads…`, `TestReadNoticeChild` |

**Run watch (card 3), `TestRunWatch` rows unless named**

| Behaviour | Pinned by |
|---|---|
| Line text, one line per reading | `AChildBlockedAtARowPostsItsLineOnce` |
| Child reason relative to the pair, on one line | `AReasonIsRelativeToThePair…` |
| Batten's reason as the fallback, child's reason winning | **none** (F2) |
| Child episode from folded repeats | `AFoldedRepeat…` |
| Child episode from history length; batten-side episode | **none** (F3) |
| Reason or time change alone owes nothing under `rows` | **none** (F5) |
| `stops`, `rows`, `off`; stop states | `StopsTellsOnlyTheStops`, `RowsTellsEveryChange`, `OffTellsNothing`, pause, cancel and done rows |
| Child not read without a write; failed read retried | `AChildChangedWithNoStatusWrite…`, `AFailedChildRead…` |
| A batten status write alone re-reads the child | **none** (F8) |
| Vanished child keeps its last part | `ALandedRunWithLandNoticeOff…` |
| Level never read, level kept on failure and past Land | the level rows |
| `land_notice` copy, skip and drop; cancelled never skipped | the landed, unlisted and cancelled rows |
| Anchor change holds; restart repeats nothing | `AStandingAnchorChange…`, `ARestartedDobby…` |
| Entry dropped or kept by status and owed lines | the three `AnUnlisted…` rows |
| Failed or nil `StatusExists` keeps the entry | **none** (F11) |
| Delivery order, no orch, retry, three failures, awake hour | the owed, failing and hour rows |
| Dobby log line for a posted watch line | **none** (F9) |
| Failed state record blocks the done-run clear | **none** (F4) |
| Clear waits on owed line or unread level; clear drops the entry | `TestClearDoneRuns` |

**Gate hold (cards 5 to 7), `TestGateHolds` rows unless named**

| Behaviour | Pinned by |
|---|---|
| Marker recorded; older record reads empty | `TestGateTest_DetachedRun`, `TestHolders_…` |
| Awake bound, sleep, minutes fallback, factor 0 | the matching rows |
| Largest tier timeout of a mixed hold | **none** (F10) |
| Once per hold, multi-slot, new hold | the matching rows |
| Noticed keys of ended holds are pruned | **none** (F7) |
| Noticed keys survive the run watch rewriting the map | **none** (F6) |
| Holder under a subdirectory; sibling rejected | first row, `ASiblingWorktree…` |
| Holder at the task worktree itself | **none** (F1) |
| Failed holders, config, tree, marker scan, worktree, post | the matching rows |
| Descendant cap and CPU order | `ALongProcessListIsCutAndCounted` |
| Watch level `off` still notices | `AWatchLevelOfOff…` |
| `TierTimeout` mapping, config load error | `TestDobbyGateHolders` |
| Mapping of the other holder fields; hub `TaskWorktree` wiring | **none** (F12) |
| Watch and gate posts: stamp, tags, no supersede | `TestHubDobbyDeps_RunNoticePost…`, `TestRunNoticeTags` |
| `RunWatch.Level` wiring | `TestHubDobbyDeps_LandNoticeReads…` |

## Findings

**F1, MEDIUM, design.** `internal/dobbyengine/gatehold.go:270` (`underWorktree`).
- No test has a holder whose `Worktree` equals the task worktree, although this is the production case (`internal/gatecli/run.go:828` records `location.WorktreePath()`).
- Dropping `path == worktree ||` passes every row and disables the notice for ordinary gate runs.
- Evidence: `gatehold_test.go` uses only `/hub/task/sub` (line 48) and `/hub/task-b`.
- Fix: add a row, or move the first row's second slot to `/hub/task`.

**F2, MEDIUM, design.** `internal/dobbyengine/runwatch.go:277-280`.
- The batten reason fallback and the child-over-batten precedence have no row, since no `RunView` in `runwatch_test.go` sets `Error`.
- Removing `reason = reading.BattenError`, or dropping `BattenError: run.Error` at line 218, passes. A batten block with no child reason would then mail no reason (D3).
- Fix: give `BattenBlockedAtLandOverADoneChild…` an `Error` and assert it, plus one row where both reasons are set.

**F3, MEDIUM, design.** `internal/dobbyengine/runwatch.go:235, 240-245`.
- Only the child's folded repeats are varied (`AFoldedRepeat…` holds `HistoryLen` at 3).
- No row has batten blocked twice at one row with a higher `run.HistoryLen` or `LastRepeats`.
- Deleting the `run.State != runStateRunning` case, or dropping `child.HistoryLen` from the child's sum, passes. D1's second-stop detection is then lost for batten stops and for non-folded child re-blocks.
- Fix: add both rows.

**F4, MEDIUM, design.** `internal/dobbyengine/donerun.go:53`, `runwatch.go:146-150`.
- Nothing tests that a watch step which cannot read or write the record sets `watchRecordFailed`, or that `clearDoneRuns` then clears no run. Card 3 asks for this hold.
- Removing `d.watchRecordFailed ||` passes, and a landed run can be cleared before its line is recorded.
- Fix: add a `TestClearDoneRuns` case with the flag set by a real unwritable or unreadable record.

**F5, LOW, design.** `internal/dobbyengine/runwatch.go:65-70` (`compared`).
- No `rows` row rewrites a status with only a new time or reason text.
- Stripping nothing from `Since`, `BattenError` or `ChildError` passes, and `rows` would mail a line on every status rewrite.
- Fix: add a `rows` row calling `writeChild` with the same state and row and a different error.

**F6, LOW, design.** `internal/dobbyengine/runwatch.go:116-139` with `gatehold.go:127-135`.
- Every once-per-hold row runs with nil `RunWatch` seams, so `runWatch` returns early.
- `AWatchLevelOfOff…` ticks once. Nothing fails if the watch's whole-map rewrite loses `GateHolds` and the notice repeats on each probe tick.
- Fix: tick twice, with a restart, in the row that wires `RunWatch`.

**F7, LOW, design.** `internal/dobbyengine/gatehold.go:107`.
- No row asserts that the key of an ended hold leaves `GateHolds`. Removing the `DeleteFunc` prune passes, and the state record grows without bound for a live run.
- Fix: assert the record's keys in `AnotherProcessOrALaterStartIsANewHold`.

**F8, LOW, design.** `internal/dobbyengine/runwatch.go:224`.
- No row changes only `run.StatusModified` and expects a child re-read (D1: "batten's or the child's status file").
- Dropping that clause passes.
- Fix: add the case to `AChildChangedWithNoStatusWriteIsNotRead`.

**F9, LOW, design.** `internal/dobbyengine/runwatch.go:320`.
- The `run watch <slug>: <line>` dobby log line has no assertion. `donerun_test.go` even nils `deps.Log` around `runWatch`.
- Fix: use `recordLog()` in one `TestRunWatch` row.

**F10, LOW, design.** `internal/dobbyengine/gatehold.go:197-203` (`gateHoldBound`).
- Every multi-slot hold in the tests has equal timeouts, so taking `hold[0].TierTimeout` instead of the maximum passes.
- Fix: give the two-slot row differing timeouts.

**F11, NIT, design.** `internal/dobbyengine/runwatch.go:132-137`.
- Card 3's "a failed or nil `StatusExists` keeps it" has no row for the failed case.
- Fix: add one beside the `AnUnlisted…` rows.

**F12, MEDIUM, design.** `internal/beamcli/dobbydeps.go` (`dobbyGateHolders` mapping; `GateHolds.TaskWorktree` at lines 331-337).
- `TestDobbyGateHolders` asserts only `Tier` and `TierTimeout` (`dobbydeps_integration_test.go`, the `got[holder.Tier]` map), and the engine tests use a fake.
- Dropping `Worktree`, `PID`, `Started`, `Marker`, `Slot` or `Site` from the mapping passes every test.
- No test calls the wired `deps.GateHolds.TaskWorktree` or `Holders`, so a holder's recorded worktree is never matched against the resolved one.
- Fix: assert the whole mapped holder, and compare `deps.GateHolds.TaskWorktree(slug)` with a real pair's path.

Tool calls: none.
