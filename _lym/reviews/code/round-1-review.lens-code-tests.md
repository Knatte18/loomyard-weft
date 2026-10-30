```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour, the test that pins it, or none.

**Per behaviour**

| Behaviour | Pinning test |
|---|---|
| Dead driver, text, relative paths, respawn-cap text | `TestRunNotice_Conditions` rows `ADeadDriver…`, `…AtTheRespawnCap…` |
| Covered/paused child and not-running child send nothing | `…ACoveredChildSendsNothing`, `…AChildThatIsNotRunningSendsNothing` |
| Quiet window from newest run, inside window, live wait holds quiet | `IdleAgents…`, `ALiveWaitHoldsTheQuietNotice` |
| No-run quiet fallback to the newest history entry | `NoRunReadFallsBack…` and its inside-window row |
| Precedence usage-limit > api-error > quiet, reset-unknown text, idle time | `AnAPIError…`, `AUsageLimitOutranks…`, `AUsageLimitWithNoReadableReset…` |
| Unreadable agents send nothing | `AnUnreadableActivitySendsNothing` |
| Idle clocks subtract pauses and sleeps | `APausedInterval…`, `ASleepIsNotIdleTime`, `TheAwakeIdleTimeAfterASleep…` |
| Once per episode, marker write, restart no-resend, new episode drops key | `TestRunNotice_OncePerEpisode` |
| Revive on first sight, notice on next sight, episode ending, streak of 3, status write resets | `TestRunNotice_APIErrorRevive` |
| Retry on probe ticks, drop after 3, nothing marked for a failed post | `TestRunNotice_DeliveryIsRetried…` |
| Status commit carries the marker only when it exists | `TestBattenRunCommitPaths` |
| Hub wiring fills every seam, quiet is 30m, marker path | `TestHubDobbyDeps_PrimeTidyShuttle` |
| Absent task worktree reads as not found; `Agents` refuses it | `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` |
| `RespawnCapReached` | `innerrun_respawncap_test.go:91` |
| Batten row sends nothing for a running child | `TestNotice_ARunningChildSendsNoNotice` |
| Batten state-changed, driver-not-revived, restart, done, awaiting and delivery notices | the retained `TestNotice_*` tests |

**Findings**

- **F-T1 — MEDIUM, design.** `internal/dobbyengine/runnotice.go:379` (`saveNoticeMemory`) and `internal/dobbyengine/state.go:66` (`RunState.Notice`). No test pins the notice memory surviving a dobby re-exec through the state record.
  - `TestRunNotice_OncePerEpisode` replaces the `daemon`, but its driver-dead key is recomputed from `child.Modified`, so that test passes with `saveNoticeMemory` removed.
  - The api-error revive streak (`APIReviveKey`, `APIRevives`, `APIReviveStatusTime`) is never checked across a restart. The acceptance criteria and the focus both name this.
  - Fix: add a restart row to `TestRunNotice_APIErrorRevive`. Tick after the first revive, replace the daemon, then assert no second revive and a notice at the next sight. Also assert the persisted `RunState.Notice` fields.
- **F-T2 — MEDIUM, design.** `internal/dobbyengine/runnotice.go:149-168` (`runNotices` selection). No test covers the run selection.
  - Uncovered: a run with a cancel marker or a non-running batten state is skipped; a child with no status file (`found=false`) sends nothing; `ReadChild` returning an error sends nothing and does not fail the tick; a nil `ReadChild` sends nothing.
  - The `AChildThatIsNotRunning…` row sets the child's state, not the run's batten state. Its `ReadChild` always returns found.
  - D2 names "an absent child status or a cancel marker" as in no condition.
  - Fix: add table rows using `runningRun` variants with `CancelMarker` and a non-running `State`, plus `ReadChild` fakes returning `found=false` and an error.
- **F-T3 — LOW, scope.** `internal/dobbyengine/runnotice.go:279`, `:310`. `Quiet == 0` ("zero sends none") has no row.
  - Fix: add a row to `TestRunNotice_Conditions` with `h.d.deps.RunNotices.Quiet = 0` and a long idle time, expecting none.
- **F-T4 — LOW, scope.** `internal/battencli/dobbynotice.go:146-214` (`readNoticeChild`, `noticeDriverAlive`). Only the absent-worktree path of `readNoticeChild` is tested.
  - Uncovered: mapping of `Modified`, `NewestHistory`, `HistoryLen` and `RespawnCapped` from a real status file and memory record. The driver-alive fallback is also uncovered: a held driver or run lock reads as alive, and an unreadable lock reads as alive.
  - These are moved reads, and the older `driverAliveFrom` tests cover the strand half.
  - Fix: add a `readNoticeChild` test over a temp task worktree with a status file, and a `noticeDriverAlive` row set for lock-held and lock-unreadable.
- **F-T5 — NIT, scope.** `internal/dobbyengine/runnotice.go:169-173`. The `deliveries` cleanup for runs no longer listed has no test. The retry test would not fail if it were removed.

**Tool calls (2)**
1. Listed test names and the diffs of the battencli, beamcli and shedrecipe tests; the manifest did not include the test files.
2. Read `internal/dobbyengine/runnotice_test.go` in full; the manifest lacked it.
3. Grepped test usage of `RespawnCapReached`, `readNoticeChild`, `noticeDriverAlive` and `DobbyRunNotices`, and the retained `battenshed` `TestNotice_*` names, to confirm batten-side coverage.
