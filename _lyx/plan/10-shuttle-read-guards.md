# Card 10 — Shuttle's polled failures and holds log once per episode

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
- `internal/shuttleengine#TestSend_WaitsForIdleSession`
- `internal/shuttleengine#TestGatePending_ExpiryProbeEndsMidTurnWait`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`
- `internal/shuttleengine#TestGate_UnsentRepromptRetries`

**Intent:**
D11 for `internal/shuttleengine`: the four sites the audit in `00-overview.md` lists for this package as guard or demote.

Operator activity: `operatorActivityHold` is asked on every idle-wait poll and every orch idle probe, and its `shuttle: operator activity unreadable, not holding the write` Warn repeated on each while tmux could not list its clients.
`Runner` gains an `activityFailures logger.EpisodeSet` and `sendContext` an `activityFailures *logger.EpisodeSet`, which both `newSendContext` constructors point at their runner's set, so every send and probe of one runner shares one episode per strand.
`operatorActivityHold` takes the set: a query error calls `failures.Fail(guid, …)` with the line's message and fields, and a successful query calls `failures.Recover(guid, "shuttle: operator activity readable again", "strandGUID", guid)`; a nil set logs the Warn as today.
Its callers `Runner.SessionIdle`, `awaitIdleSession`, `busyReading`, `clearStrayCharacter` and `guardedEnter` pass `&r.activityFailures` or `sc.activityFailures`.

`Run` gains a `failures logger.EpisodeSet`, living as long as the run's wait:
`probeExpiry`'s `shuttle: gate: expiry probe failed, treating the entry as not expired` Warn, repeated on every gate re-check while an entry's probe errors, becomes `run.failures.Fail("expiry:"+entry.Name, …)`, and a probe answer without an error calls the matching `Recover` with `shuttle: gate: expiry probe answers again`;
`checkLivenessTick`'s `shuttle: capture pane during startup probe (non-fatal, retrying)` Warn, repeated on every startup probe while the capture fails, becomes `run.failures.Fail("startup capture", …)`, and a successful capture calls `Recover` with `shuttle: startup probe capture works again`.
`handleGatedBoundary`'s `shuttle: gate: re-prompt not delivered, retrying on a later tick without spending an attempt` is a held send, an ordinary retry iteration rather than a failure, so it logs at Info when `run.unsentReprompt` was false before this attempt and at Debug while it stays set.

Tests, each extended where it already reaches the site:
`TestSend_WaitsForIdleSession` gains a `wantActivityWarns` column, 1 for its two `activityErr` rows, each of which builds its own fixture and so its own runner: the count of `level=WARN` `operator activity unreadable` lines across the row's polls.
`TestGatePending_ExpiryProbeEndsMidTurnWait`'s "probe error warns and keeps polling" row asserts exactly one `level=WARN` `expiry probe failed` line across its re-checks.
`TestRun_Wait_StartupWindow`'s two "pane capture fails every probe" rows assert exactly one `level=WARN` `capture pane during startup probe` line across the window's probes.
`TestGate_UnsentRepromptRetries` drops its row-level `t.Parallel` so each row can capture the process-global logger, and asserts one `level=INFO` `re-prompt not delivered` line and no WARN one for "a busy re-prompt is re-sent once the pane idles" and "a re-prompt that stays undelivered is retried until the run deadline".
Each of these captures at Info through `logcapture.CaptureVerbose`; a test that already captures keeps its own capture.

**ImpactSummary:** one re-signed unexported helper whose five callers are listed here, two new fields on `Runner`/`Run` and one on `sendContext`; log levels and repetition change, and no hold, send or outcome decision does.

**Commit:** 10: shuttle: log polled failures and held re-prompts once per episode
