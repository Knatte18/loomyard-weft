# Card 8 — Batten waits log Info once per state; Warn means a failed step

**Create:**
- `plan:internal/battenshed#reportWait` -> `func reportWait(producer, reason, scratchDir string, fields ...any)`
- `plan:internal/battenshed#writeStuckReason` -> `func writeStuckReason(producer, reason, scratchDir string)`
- `plan:internal/battenshed#innerRunProducer.failedEnd` -> `func (p *innerRunProducer) failedEnd(text string) *waitEnd`
- `internal/battenshed/stuck_test.go`
- `plan:internal/battenshed#TestStuckReport_LevelFollowsCause` -> `func TestStuckReport_LevelFollowsCause(t *testing.T)`

**Edit:**
- `internal/battenshed#reportStuck`
- `internal/battenshed#innerRunProducer.exemptEnd`
- `internal/battenshed#innerRunProducer.report` -> `func (p *innerRunProducer) report(w *childWait, reason, key string)`
- `internal/battenshed#childWait`
- `internal/battenshed#innerRunProducer.checkChild`
- `internal/battenshed#innerRunProducer.runArm`
- `internal/battenshed#innerRunProducer.holdSpawnWhileCovered`
- `internal/battenshed#innerRunProducer.publishRetryStep`
- `internal/battenshed#innerRunProducer.publishRetryGoto`
- `internal/battenshed#innerRunProducer.publishRetryResume`
- `internal/battenshed#innerRunProducer.staleBaseResume`
- `internal/battenshed#innerRunProducer.runningStep`
- `internal/battenshed#innerRunProducer.haltedStep`
- `internal/battenshed#innerRunProducer.awaitingStep`
- `internal/battenshed#innerRunProducer.resumeAwaitingChild`
- `internal/battenshed#innerRunProducer.doneStep`
- `internal/battenshed#innerRunProducer.warnHaltEpisode`
- `internal/battenshed#innerRunProducer.logSessionStatesAtNotice`
- `internal/battenshed#landProducer.stuck`
- `internal/battenshed#TestInnerRun_RunningArmReviewWaitNote`
- `internal/battenshed#TestInnerRun_DoneFinishesPastGraceWithLiveDriverAndWarns`
- `internal/battenshed#TestInnerRun_HaltWarnsOncePerEpisode`
- `internal/battenshed#TestInnerRun_ReHaltAfterObservedResumeWarnsAgainAtTheSameHistoryLength`
- `internal/battenshed#TestNotice_LogsEachRunsSessionStateAtAQuietOrAPIErrorFinding`
- `internal/battenshed#TestInnerRun_TerminalOpenErrorIsWarnedNotEscalated`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed/doc.go`

**Intent:**
D5, D6 and D7: a batten log line about a stuck verdict takes its level from its cause, a wait logs once per state of the child, and the two smaller lines move down.

The reason file and the log line split:
`writeStuckReason` is `reportStuck`'s file half, unchanged (create the scratch directory, overwrite `<producer>-stuck.md`, Warn on a write failure, return nothing);
`reportStuck` keeps its signature and logs `battenshed: producer stuck` at Warn before calling it, and is now the route of a failed producer step only, which its existing callers in `create.go`, `seamchild.go`, `land.go` (`stuck` with `notify` set) and `teardown.go` already are;
`reportWait` logs the same message and fields at Info and calls `writeStuckReason`, and is the route of an ordinary wait or move.
`landProducer.stuck` calls `reportWait` for a pause (`notify` false) and `reportStuck` otherwise.
`exemptEnd` now calls `reportWait`, since its callers end the wait on a state change, a requested pause or a Publish-retry action;
the new `failedEnd` is the same verdict through `reportStuck`, and `checkChild`'s two status-file ends (cannot be read, gone) call it.

`report` takes a dedupe key beside the text: it returns when `reason` equals `w.reason`, else records it and calls `writeStuckReason`, so the reason file always holds the newest text for `lyx batten status`;
it logs `battenshed: producer stuck` at Info, with the full reason, only when `key` differs from `childWait`'s new `reasonKey` field, which it then records.
Each caller passes the reason with its counters, countdowns and error tails left out, never parsed back out of the text:
`runArm`, `holdSpawnWhileCovered`, `staleBaseResume`, `runningStep`, `haltedStep`, `awaitingStep` and `resumeAwaitingChild`'s not-parked reason pass the reason itself, which carries none;
`doneStep` passes `inner shed run is done; waiting for its driver to finish its stop report`, the reason without its elapsed and grace counts;
`resumeAwaitingChild`'s busy reason passes a new `childWait.busyKey`, set beside `busyReason` to the same text without its parenthesised spawn-error tail, which also keeps the driver pane's draft text out of the key;
`publishRetryStep`, `publishRetryGoto` and `publishRetryResume` pass a new `childWait.publishRetryKey`, set beside `publishRetryReason` to the same text without its parenthesised refusal tail.
A reason whose text only counts up then logs once per state of the child.

D6: `warnHaltEpisode` logs `battenshed: inner shed run halted; waiting for the operator to resume it` at Info, still once per halt episode through the halt-warned marker; the method keeps its name.
D7: `logSessionStatesAtNotice` logs `battenshed: session state at notice` at Debug.
`stuck.go`'s header and `reportStuck`'s doc comment describe the split; `NewInnerRun`'s doc comment, whose arm table says the blocked, paused or failed arm "Warns once per halt episode", says it logs one Info line per halt episode, and so does `innerrun_halted_test.go`'s file header comment, edited with the two tests of that file this card lists; and `doc.go`'s Run-Shed account says a halt logs one Info line per episode where it says Warn, and that waits log at Info once per state while a failed step logs Warn.

Tests:
`TestStuckReport_LevelFollowsCause`, in the new `stuck_test.go`, is serial (it captures the process-global logger at Info through `logcapture.CaptureVerbose`) and table-driven, one row per cause, each driving a producer through the fixture an existing test of that producer already uses.
Each row names the reason text its cause produces and asserts that the one `battenshed: producer stuck` line carrying that reason has the row's level; a line an earlier wait logged before the row's cause, such as the running arm's entry report, is not the row's line and is not asserted.
The INFO rows, the new `reportWait` route, also assert that `StuckReasonFile(scratchDir, name)` holds the row's reason; the WARN rows' file writes are what the producers' own tests already pin, so those rows assert the level only.
WARN rows: a Worktree-Create whose create seam fails (built as `TestWorktreeCreate_CreateErrorsStickWithTheirTextVerbatim` builds it), a Seed-Child whose board read fails (as `TestSeedChild_UnreadableBoardIsStuck`), a Land whose pull fails (`landFixture` with `pullErr`), a Worktree-Teardown whose shutdown fails (as `TestWorktreeTeardown_ShutdownFailureNeverCallsRemove`), and an InnerRun whose status file goes missing mid-wait (`newInnerRunDeps` with a running status followed by a scripted not-found `ReadStatus` answer on a probe check, so `checkChild` reaches `failedEnd` after the running arm's entry report);
INFO rows: an InnerRun whose child moves from running to awaiting (a state change), one whose child is blocked and whose wait ends on a pause request (`fakeClock.pauseAtSleep`), a Land paused in its land-lock wait (`landFixture` with `landLockBusy` and `pauseAfterSleeps`), and an InnerRun whose automatic Publish verify retry is refused busy before a pause ends its wait (built as `innerrun_publishretry_test.go`'s busy-goto row builds it).
`TestInnerRun_DoneFinishesPastGraceWithLiveDriverAndWarns` captures at Info and records reasons through `fakeClock.watchReason`, and additionally asserts that the done arm's reason took more than one text over the grace while exactly one `inner shed run is done; waiting for its driver` line was logged, and that the reason file holds the last recorded text.
`TestInnerRun_RunningArmReviewWaitNote` captures at Info, since its counted lines are Info now; its write counts are unchanged, since each running reason is its own key.
`TestInnerRun_HaltWarnsOncePerEpisode` and `TestInnerRun_ReHaltAfterObservedResumeWarnsAgainAtTheSameHistoryLength` capture at Info and assert their halt-line counts at `level=INFO`.
`TestNotice_LogsEachRunsSessionStateAtAQuietOrAPIErrorFinding` captures at Debug (`logger.SetVerbosity(2)` after its capture), and adds that no `level=INFO` line carries `session state at notice`.
`TestInnerRun_TerminalOpenErrorIsWarnedNotEscalated` keeps its relative count, which still holds, and its comment that the running arm's stuck line is a warning in both runs says that line is Info now, so the failed open's Warn is the only one either run adds.
The package's 109 top-level tests become 110, within its budget row of 110.

**ImpactSummary:** `report` is re-signed with every caller listed here; `reportStuck`'s callers keep their call and their Warn, and the only observable changes are the log levels and the one-line-per-state dedupe, while every reason file write and every verdict stays as it was.

**Commit:** 8: battenshed: waits log Info once per state, failed steps Warn
