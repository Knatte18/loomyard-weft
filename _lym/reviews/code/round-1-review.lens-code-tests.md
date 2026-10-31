```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds or changes, the test that pins it or none, from the test diffs and three extra greps.

**Behaviour → pinning test**

- `PastLand`, `LandOutcomeOf` (D1): `TestLandOutcomeOf` rows (`land_test.go`). They cover ahead, past with Land Done, past without it, and teardown in any state. Pinned.
- Hibernate trigger calls `PastLand`, `landFinished` removed (D1): no test added or changed, which fits "behaviour unchanged". I did not check whether existing hibernate `after` tests still exercise the trigger. Not verified.
- `land_notice` default, `all`, `off`, refusal naming `lym config beam --set land_notice=` (D2): `TestLoadConfig` rows (`config_test.go`). Pinned. No row for an explicit `per-task`; the template-default row covers it.
- `notify_on_land` stored only when true, set on task and project, false clears, note refused, `ClearNotifyOnLand` clears or writes nothing (D3): `TestNotifyOnLand` (`board_test.go`), including the byte-identical checks. Pinned.
- Run lock ignores the field (D3): `TestRunLock` row "notify_on_land change". Pinned.
- Revised date ignores the field (D3): none. The runlock row checks acceptance on a held entry, not the date. No other test touching the date was found in the diff. **Finding 1.**
- Board replay carries the field (D3): none. `replayFields` has the new row, but no replay test was added and the grep of `replay_test.go` found nothing. **Finding 2.**
- Every upsert path accepts the field (D3): the `TestNotifyOnLand` rows and the `help_test` schema row cover `UpsertTask` and the help text. The batch and other upsert paths have no row. **Finding 3.**
- Landing-notice state machine (D4, D5): `TestLandNotices` (`landnotice_test.go`) covers these rows:
  - `all`, `per-task` requested or not, and `off`;
  - first seen past Land sends nothing;
  - teardown row counts as past Land;
  - the unlanded line and the commit-unknown line;
  - the 12-character commit;
  - the request kept after the prune and cleared after the send;
  - retry on the next tick and drop after three attempts;
  - a run no longer listed is forgotten;
  - the memory is empty once settled.
- Not covered in `TestLandNotices`:
  - the `Requested` refresh while the entry is on the board, including a flip true→false or false→true before Land; only one fixed value per row is tested;
  - the board read failing (`Tasks` error) so the last `Requested` is kept;
  - the entry gone from the board before Land, so the last `Requested` is kept (the focus question);
  - the `LoadConfig` failure leaving the memory for a later tick;
  - the `ClearNotifyOnLand` failure only being logged, and a nil seam;
  - a commit shorter than 12 characters (the `min` branch).
  The refresh, the board-read failure and the gone-before-Land case are **Finding 4**. The `LoadConfig` failure is **Finding 5**. The clear failure and nil seam are **Finding 6**.
- `ClearNotifyOnLand` gated on `requested` (D5): the `PerTaskRequestedSendsAndClears…` and `AllRequestedClearsTheRequest` rows pin it. Pinned.
- `landNotices` runs on each tick after the passes, from `listedRuns` with done runs included (D5): the rows list done runs (`landed` has `State` done) through `h.tick`. The tick-level call site in `loop.go` and the no-run-when-anchor-change-stands rule have no test. `loop_test.go` only gains a `Tasks` stub. **Finding 7** (LOW).
- `ViewRun`, `WaitingTask` and the `land_notice` value (D7): `TestBuildView` row "TheLandNoticeValueAndEachRequestShow". It covers the on-board, remembered and plain cases and a waiting task. Pinned.
- `DobbyLines` header and `notify_on_land` suffix (D7): `TestDobbyLines` rows. Pinned. The `lym beam status` text reuses `DobbyLines` and has no separate test, which is reasonable.
- `DobbyTasks` reads the field: `TestDobbyTasks_ReadsDependenciesParentAndNotifyOnLand`. Pinned.
- `DobbyRuns` outcome and commit (`shedrun/landed.go`, `dobbyreads.go`): `TestDobbyRuns_ReadsTheLandOutcomeAndTheLandingCommit` (tier1) covers ahead, landed with commit and unlanded. Pinned. Missing or unreadable commit file yielding "unknown" has a read-side test only for the missing file (`TestLandedCommit` before a write). An unreadable file is untested, which is a low-value gap.
- `shedrun` path and round trip: `TestLandedCommit`. Pinned.
- `Finalize` records the head after the board mark and before the push on the retry-landing path: `TestFinalize_MergeInRequiredRetry` rows. Pinned for the retry site only.
- `Finalize` records on the first-try merge success (`finalize.go` first `recordLanding` site): no test asserts a recorded commit there. `TestFinalize_RecordLandingFailureKeepsDone` uses a first-try merge but its seam discards the commit. The "already-landed" path, if separate from the retry, is also unasserted. **Finding 8.**
- Nothing recorded when the merge fails: `TestFinalize_StuckReasons`, with `RecordLanding` setting `markCalled`. Pinned.
- A failed head read or record keeps Done and pushes: `TestFinalize_RecordLandingFailureKeepsDone`. Pinned.
- `loomcli` `RecordLanding` wiring (`landingdeps.go`): none. Nothing pins that the write path (`primeOfHub`, `seedSlug(l.WorktreeName)`) names the same file as the read side (`battencli` `DobbyRuns` with `slug`). **Finding 9** (MEDIUM, the focus asks for this).
- `beamcli` `ClearNotifyOnLand` wiring (`dobbydeps.go`): none. No test file mentions it besides the engine and board tests. **Finding 10** (LOW).
- `Landing.RecordLanding` in `envkit` `nilLegal`: no test needed, since it is a table entry.

**Findings**

1. LOW, design. The revised date ignoring `notify_on_land` has no row. Evidence: `board_test.go` `TestNotifyOnLand` and `runlock_test.go` row, no revised assertion.
2. LOW, design. Replay carrying `notify_on_land` is untested. Evidence: `replay.go:54`, no replay test in the diff.
3. LOW, design. Only `UpsertTask` is exercised for "accepted by every upsert path". Evidence: `board_test.go` `TestNotifyOnLand` acts.
4. MEDIUM, design. The `Requested` refresh, a board-read failure keeping the last reading, and an entry gone before Land have no row. These are the focus's "pruned before Land" case and the `UpdateState` closure's use of `requests`. Evidence: `landnotice.go` `landNotices`; `landnotice_test.go` sets `tasks` only to a fixed `tt.requested`.
5. LOW, design. A `LoadConfig` failure on a due tick, which must leave the memory so a later tick settles it, has no row. Evidence: `landnotice.go` `landNotices` after the closure.
6. LOW, design. A failed `ClearNotifyOnLand`, which is only logged, and a nil seam have no row. Evidence: `landnotice.go` `settleLandNotice`.
7. LOW, design. The `tick` call site (after the passes, skipped while an anchor change stands) is unpinned. Evidence: `loop.go` `tick`; `loop_test.go` diff is a stub only.
8. MEDIUM, design. `Finalize` recording the head after a first-try parent-side merge has no assertion of the recorded value or order. Evidence: `finalize.go` first `recordLanding` call; `finalize_test.go` `TestFinalize_RecordLandingFailureKeepsDone` ignores the commit.
9. MEDIUM, design. The `loomcli` write side is not tied to the read side. Evidence: `landingdeps.go` `RecordLanding`; `dobbypairanchor_test.go` writes via `shedrun.WriteLandedCommit` directly.
10. LOW, design. `hubDobbyDeps`'s `ClearNotifyOnLand` closure is unpinned. Evidence: `beamcli/dobbydeps.go`.

**Tool calls**
1. `git diff … -- '*_test.go'`: the test diffs were not in my context.
2. `sed`/`grep` over the saved output: to list test names (same file as call 1).
3. `sed` over the saved output: to read the dobby, board and finalize test bodies.
4. `grep` of `replay_test.go`, `git diff` of `overviewdobby_test.go`, and `grep -rln` for `RecordLanding|ClearNotifyOnLand` in tests: to check replay, overview and wiring coverage.
