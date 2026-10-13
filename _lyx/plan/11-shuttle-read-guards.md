# Card 11 — Shuttle's polled failures and holds log once per episode

**Edit:**
- `internal/shuttleengine#Runner`
- `internal/shuttleengine#sendContext`
- `internal/shuttleengine#Run.newSendContext`
- `internal/shuttleengine#Runner.newSendContext`
- `internal/shuttleengine#operatorActivityHold` -> `func operatorActivityHold(reed ReedOps, guid string, failures *logger.EpisodeSet) string`
- `internal/shuttleengine#Runner.SessionIdle`
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#busyReading`
- `internal/shuttleengine#clearStrayCharacter`
- `internal/shuttleengine#guardedEnter`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.probeExpiry`
- `internal/shuttleengine#Run.handleGatedBoundary`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#Run.watchDialog`
- `internal/shuttleengine#ReadAgentWaits`
- `internal/shuttleengine#heldMarkerLive`
- `internal/shuttleengine#ReadAgentActivity`
- `internal/shuttleengine#newestTurnEnd`
- `internal/shuttleengine#ReadWaitMarkers`
- `internal/shuttleengine#ReadSessionStates`
- `internal/shuttleengine#TestReadAgentWaits`
- `internal/shuttleengine#TestReadAgentActivity`
- `internal/shuttleengine#TestReadWaitMarker`
- `internal/shuttleengine#TestReadSessionStates`
- `internal/shuttleengine#TestSend_WaitsForIdleSession`
- `internal/shuttleengine#TestGatePending_ExpiryProbeEndsMidTurnWait`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`
- `internal/shuttleengine#TestGate_UnsentRepromptRetries`
- `internal/shuttleengine#TestGatePending_FailedSendWarnsAndStaysPending`
- `internal/shuttleengine#TestWait_DialogNotice`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeReed.OperatorActive`
- `internal/shuttleengine#fakeReed.SendKey`

**Intent:**
D11 for `internal/shuttleengine`: the sites the audit in `00-overview.md` lists for this package as guard or demote.

Operator activity: `operatorActivityHold` is asked on every idle-wait poll and every orch idle probe, and its `shuttle: operator activity unreadable, not holding the write` Warn repeated on each while tmux could not list its clients.
`Runner` gains an `activityFailures logger.EpisodeSet` and `sendContext` an `activityFailures *logger.EpisodeSet`, which both `newSendContext` constructors point at their runner's set, so every send and probe of one runner shares one episode per strand.
`operatorActivityHold` takes the set: a query error calls `failures.Fail(guid, …)` with the line's message and fields, and a successful query calls `failures.Recover(guid, "shuttle: operator activity readable again", "strandGUID", guid)`; a nil set logs the Warn as today.
Its callers `Runner.SessionIdle`, `awaitIdleSession`, `busyReading`, `clearStrayCharacter` and `guardedEnter` pass `&r.activityFailures` or `sc.activityFailures`.

`Run` gains a `failures logger.EpisodeSet`, living as long as the run's wait:
`probeExpiry`'s `shuttle: gate: expiry probe failed, treating the entry as not expired` Warn, repeated on every gate re-check while an entry's probe errors, becomes `run.failures.Fail("expiry:"+entry.Name, …)`, and a probe answer without an error calls the matching `Recover` with `shuttle: gate: expiry probe answers again`;
`checkLivenessTick`'s `shuttle: capture pane during startup probe (non-fatal, retrying)` Warn, repeated on every startup probe while the capture fails, becomes `run.failures.Fail("startup capture", …)`, and a successful capture calls `Recover` with `shuttle: startup probe capture works again`;
`checkLivenessTick`'s `shuttle: dismiss trust prompt (non-fatal)` Warn, repeated on every startup probe while `playInputsSent` sends nothing, since `run.dismissed` is set only when an input was sent, becomes `run.failures.Fail("startup dismiss", …)`, and a dismissal that sends calls `Recover("startup dismiss", "shuttle: trust prompt dismissal sends again", "strandGUID", …)`.
A dismissal that sends is a `playInputsSent` call that returns no error for a non-empty sequence, the branch that logs `shuttle: dismissed startup gate`;
a call that returns an error is a failure even when `sent` is greater than 0, so a partial send calls `Fail` and never `Recover`;
`watchDialog`'s `shuttle: capture pane for the dialog watch failed, retrying on a later tick` Warn, repeated on every wait tick while a dialog latch stands and the capture fails, becomes `run.failures.Fail("dialog capture", …)`, and a capture without an error calls `Recover` with `shuttle: capture pane for the dialog watch works again`;
`handleGatedBoundary`'s `shuttle: gate: pending send failed, entry stays pending` Warn, repeated on every gate re-check (`gateRecheckEvery`) while the pending send fails, becomes `run.failures.Fail("pending:"+entry name, …)` with its existing message and fields, and a pending send that succeeds calls the matching `Recover` with `shuttle: gate: pending send delivered again`.
`handleGatedBoundary`'s `shuttle: gate: re-prompt not delivered, retrying on a later tick without spending an attempt` is a held send, an ordinary retry iteration rather than a failure, so it logs at Info when `run.unsentReprompt` was false before this attempt and at Debug while it stays set.

The file readers other packages poll log and skip an unreadable record without returning an error, so no caller's guard on a returned error ever sees them:
`ReadAgentWaits`, `ReadAgentActivity` and `ReadSessionStates` log `shuttle: skipping an unreadable run record`, `heldMarkerLive` and `ReadWaitMarkers` log `shuttle: skipping an unreadable wait marker`, and `newestTurnEnd` logs `shuttle: could not parse a run's events for its activity`.
They run on every wake of `shedverbs`' child watch through loomcli's `readAgentStates`, on every idle-watchdog wake through loomcli's `agentActivityFor`, and on every battenshed check through battencli's `readAgentActivity` and `readAgentStates` seams, so each repeated per record while the record stayed unreadable.
The package gains one package-level `unreadableFiles logger.EpisodeSet`, since the readers are free functions called from several processes' loops, keyed by the path the line names:
each of these lines becomes `unreadableFiles.Fail(path, …)` with its existing message and fields, keyed by the run directory, the marker path or the events path;
a run record read without an error calls `Recover(runDir, "shuttle: run record readable again", "runDir", runDir)`, a wait marker read that decodes or finds no file calls `Recover(path, "shuttle: wait marker readable again", "path", path)`, and an events parse without an error calls `Recover(eventsPath, "shuttle: run events parse again", "eventsPath", eventsPath)`.
Readers sharing a path share its episode, so a broken record that two readers skip in one process Warns once.

Tests, each extended where it already reaches the site:
`TestReadAgentWaits` gains a subtest, serial like the test, capturing at Debug: it writes one run directory whose `run.json` is `{` and one live run whose wait marker is `kind: [`, calls `ReadAgentWaits` three times, rewrites both files valid and calls it once more, and asserts, for each of `skipping an unreadable run record` and `skipping an unreadable wait marker`, exactly one `level=WARN` and two `level=DEBUG` lines, each DEBUG with `failed_ticks=`, and then one `level=INFO` `run record readable again` and one `level=INFO` `wait marker readable again` line.
`TestReadAgentActivity` gains a subtest that runs `ReadAgentActivity` four times over one live run on one `fakeEngine` whose `ParseEventsFailCount` is 3, and asserts one `level=WARN` and two `level=DEBUG` `could not parse a run's events` lines and one `level=INFO` `run events parse again` line.
`TestReadWaitMarker` gains the same fail-three-then-heal subtest for `ReadWaitMarkers` over an undecodable marker, asserting one WARN, two DEBUG and one INFO line.
`TestReadSessionStates` drops its top-level and row-level `t.Parallel` so it can capture the process-global logger, and gains the same fail-three-then-heal subtest for an undecodable `run.json`, asserting one WARN, two DEBUG and one INFO line.
`TestSend_WaitsForIdleSession` gains a `wantActivityWarns` column, 1 for its two `activityErr` rows, each of which builds its own fixture and so its own runner: the count of `level=WARN` `operator activity unreadable` lines across the row's polls.
`fakeReed` gains `ActivityErrQueue []error`, consumed FIFO by `OperatorActive` with its last entry sticking, like `CaptureErrQueue`: a non-nil entry fails that call and a nil entry answers from `ActiveQueue`, while `ActivityErr` keeps failing every call.
`TestSend_WaitsForIdleSession` also gains an `activityErrQueue` column passed to that field, and a new row "an activity query that reads again after failing logs its recovery" fails the first two polls and then answers, asserting one `level=WARN` `operator activity unreadable` line and one `level=INFO` `operator activity readable again` line.
`TestGatePending_ExpiryProbeEndsMidTurnWait` gains a `probeErrCalls` column bounding how many first probe calls return `probeErr` (0 meaning every call), and its "probe error warns and keeps polling" row sets it to 2, so the probe errors on its first two re-checks and then answers;
that row asserts exactly one `level=WARN` `expiry probe failed` line and one `level=INFO` `expiry probe answers again` line across its re-checks.
`TestRun_Wait_StartupWindow`'s two "pane capture fails every probe" rows assert exactly one `level=WARN` `capture pane during startup probe` line across the window's probes.
It gains `captureErrQueue` and `sendKeyErr` columns passed to the fake's `CaptureErrQueue` and `SendKeyErr`, a `wantLogs` column of exact line counts by level and text, and two rows:
"pane capture fails on the first probes then works", whose `CaptureErrQueue` fails the first two probes and then answers, asserts one `level=WARN` `capture pane during startup probe` line and one `level=INFO` `startup probe capture works again` line;
"trust dismissal fails every probe", whose script shows the trust prompt on every probe with `gateCapture` and whose fake's `SendKeyErr` is set, asserts exactly one `level=WARN` `dismiss trust prompt` line.
`fakeReed` gains `SendKeyErrQueue []error`, consumed FIFO by `SendKey` with its last entry sticking, like `CaptureErrQueue`: a non-nil entry fails that call and a nil entry succeeds, while `SendKeyErr` keeps failing every call when the queue is empty.
`TestRun_Wait_StartupWindow` also gains a `sendKeyErrQueue` column passed to that field, and a third new row "trust dismissal fails on the first probes then sends", whose script shows the trust prompt on every probe with `gateCapture` and whose `SendKeyErrQueue` fails the first two `SendKey` calls and then succeeds, with the outcome the fail-every-probe row expects, asserts exactly one `level=WARN` `dismiss trust prompt` line and one `level=INFO` `trust prompt dismissal sends again` line.
`TestGatePending_FailedSendWarnsAndStaysPending` scripts the pending result on four evaluations before the pass and captures at Debug (`logcapture.CaptureVerbose` plus `logger.SetVerbosity(2)`).
It passes `newPendingFixture` a step, run on every tick, that clears `f.reed.SendTextErr` once `f.reed.SendTextCalls` holds two failed pending sends, so a later evaluation's pending send succeeds, and its gate-call count follows the scripted evaluations.
It asserts one `level=WARN` and at least one `level=DEBUG` `pending send failed` line and one `level=INFO` `gate: pending send delivered again` line.
`TestWait_DialogNotice` drops its top-level and row-level `t.Parallel` so each row can capture the process-global logger, and its "a failed capture sends nothing on that tick and the notice follows on a later tick that reads no box" row, whose capture fails until its step at tick 12 heals it, also asserts exactly one `level=WARN` `capture pane for the dialog watch failed` line and one `level=INFO` `capture pane for the dialog watch works again` line.
`TestGate_UnsentRepromptRetries` drops its row-level `t.Parallel` so each row can capture the process-global logger, and asserts one `level=INFO` `re-prompt not delivered` line and no WARN one for "a busy re-prompt is re-sent once the pane idles" and "a re-prompt that stays undelivered is retried until the run deadline".
Each of these captures at Info through `logcapture.CaptureVerbose`, or at Debug where named, and stays serial; a test that already captures keeps its own capture.

**ImpactSummary:** one re-signed unexported helper whose five callers are listed here, two new fields on `Runner`/`Run` and one on `sendContext`, one package-level episode set over the exported file readers' skip lines, and two fields on the test fake `fakeReed`; log levels and repetition change, and no hold, send, outcome or reader result does.

**Commit:** 11: shuttle: log polled failures and held re-prompts once per episode
