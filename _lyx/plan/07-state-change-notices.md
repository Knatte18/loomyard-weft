# Card 7 — state-change-notices

**Edit:**
- `internal/battenshed/notice.go`
- `internal/battenshed/deps.go`
- `internal/battenshed/innerrun.go`
- `internal/battenshed/doc.go`
- `internal/battenshed/notice_test.go`
- `internal/battenshed/innerrun_awaiting_test.go`
- `internal/battenshed/innerrun_halted_test.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_seams_test.go`

**Uses:**
- `internal/orchengine/notice.go`
- `internal/shedrun/paths.go`

**Intent:**
D4: batten sends each state-changed notice from inside card 5's wait, once, timed on the driver's stop report, carrying `since`, `history` and the report path.

Seams (`deps.go`, wired in `battencli/wire.go`):
- `Notify` becomes `func(ctx context.Context, line string) (queued bool, err error)`, wired to `orchengine.QueueNotice`'s own answer on the prime orch's paths; a notice is recorded as sent only when `queued` is true;
- `OrchStrandRecorded func() (bool, error)`, wired to the prime orch state's recorded strand, the same fact card 6's marker tests: with none recorded batten does not call `Notify`, logs one `Warn` per episode and asks again once per `notice_probe_s` until the episode ends or the `Done` return makes its last attempt;
- `StopReport func() (path string, at time.Time, found bool, err error)`, wired to the child's park marker (`shedrun.ParkMarker` of the task worktree's `SelfRunID` run): the path it holds and the marker's file time.

Episode state, held in the producer across calls, pause returns and error returns until the child's state changes: `since` is the status file's modification time at the first sight of the child in its state (by a check inside the wait, or by the baseline stat of a call that begins in a new state); a rewrite of the file in the same state does not move it; only a batten start reads it from the file.
The episode key of a state-changed and of a driver-dead notice becomes `condition|state|since`; `keepEpisode` drops keys by the new rule (state changed, or `since` moved).
A pending notice is derived from the child's status and the episode marker of sent keys, so it outlives the call that saw the change, and it is checked on every check inside the wait: a call that returns for the state change leaves it to the wait of the call that begins in the new state, and a pause or error return leaves it pending.

When it goes:
- a parked stop (blocked, paused, failed, awaiting): once a stop report whose file time is at or after `since` exists, or the driver strand has ended (read once per `notice_probe_s`), or 3 minutes after `since`, whichever comes first; an older report never qualifies;
- a done child: on the first check that sees it done, with no report;
- an `awaiting` child whose status carries a non-empty `ParentNotice`, while this batten holds the watched marker (card 6): no notice while the driver's relay can still happen; it goes when the driver strand has ended, or the 3 minutes passed with no report, or the child is still awaiting with no decision record 10 minutes after the report's file time. Without the marker, or with no `ParentNotice`, it goes like any parked stop.
The 3-minute report wait and the 10-minute relay wait are package constants.

Content: after today's slug, condition, state and error, every notice (state-changed, driver-dead, quiet, api-error) carries `since <RFC 3339 time>` and `history <n>` (the child's history length); a state-changed notice for a parked stop carries `report <path>`, or `report none yet` when the 3 minutes passed without one; a done notice carries no report.
The line keeps the `[batten notice]` prefix, stays one line, and the attach command stays.

Delivery: a `Notify` error is retried at most once per `notice_probe_s` and at most 3 times, then dropped with a `Warn`.
The `Done` return makes one last send attempt of a pending notice first; one not queued then is logged once and lost.
A batten started again sends the notices not yet sent, since the marker records only sent ones.

Bound, in `doc.go`: the report wait delays a notice by at most 3 minutes and the relay wait by at most 10 minutes after the report, never losing it; one notice per episode as before.

Tests (`notice_test.go` table rows and arm tests on the injected clock).
The existing `TestNotice_StateChangeNotifiesOncePerEpisode`, `TestNotice_StatusChangeStartsNewEpisode`, `TestNotice_RestartDoesNotRenotify` and `TestNotice_NotifyErrorDoesNotChangeOutcome` keep covering once-per-episode, a new episode on a state change, no re-notice after a restart and an error not changing the outcome;
they change only for the new `Notify` signature, the `condition|state|since` key, the sent-only-when-queued rule and the capped retry, and so do `TestNotice_ReturnToRunningStartsNewEpisode`, `TestNotice_DeadDriverWhileRunning` and `TestNotice_NilNotifyRunsNoStep`;
`TestRenderNotice`, the table that pins the rendered line, gains rows for `since`, `history`, `report <path>`, `report none yet` and a done notice with no report, and the cases below add only what these do not cover:
- a halted child whose driver writes a report gets one notice with `since`, `history` and that path, sent by the wait of the call that begins in the new state; with no report, one notice 3 minutes after the change with `report none yet`; a report older than `since` is never carried;
- a paused or failed child takes `since` and its 3 minutes from the status file's time at first sight; a same-state rewrite between two calls moves neither `since` nor the key and sends nothing more;
- a done child's notice goes on first sight with no report, and the `Done` return attempts a pending one;
- an awaiting child with no `ParentNotice` gets one notice with its report; with one, none while the driver is alive, a report exists and the marker is held, and one when the driver ends, after 3 minutes with no report, or 10 minutes after the report undecided; without the marker it goes at once;
- a queued notice is recorded sent; with no strand recorded `Notify` is not called and one `Warn` is logged per episode; a failing `Notify` is retried no more often than once per `notice_probe_s`, at most 3 times; a restarted batten sends a notice not yet sent;
- `wire_seams_test.go`: `TestWire_NotifyQueuesOnThePrimesOrch` and `TestWire_NotifyWritesNothingWithoutAnOrchStrand` already cover the `Notify` wiring and the no-strand case, and gain the assertion that `Notify` reports `QueueNotice`'s answer; the new cases are that `OrchStrandRecorded` reads the prime orch state and `StopReport` reads the park marker's content and time.

**ImpactSummary:** the notice step of the InnerRun producer is rewritten and `InnerRunDeps.Notify` changes signature, its one production wiring in `battencli` updated in this card; the orch notice queue is unchanged.
