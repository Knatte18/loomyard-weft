# Card 12 — Orch: benign probe states agree, draft holds carry no draft text, polled failures log once

**Create:**
- `internal/orchengine#draftHoldReason`
- `internal/orchengine#TestWatcher_DraftHoldLogsOnceWithoutTheDraftText`

**Edit:**
- `internal/orchengine#Watcher`
- `internal/orchengine#idleStatePair`
- `internal/orchengine#Watcher.logIdleStateDisagreement`
- `internal/orchengine#heldReason`
- `internal/orchengine#Watcher.heldTyping`
- `internal/orchengine#Watcher.orchNetworkOperationInFlight`
- `internal/orchengine#Watcher.Run`
- `internal/orchengine#fakeSession`
- `internal/orchengine#fakeSession.SessionIdle`
- `internal/orchengine#TestWatcher_LogsDisagreementBetweenIdleProbeAndSessionState`
- `internal/orchengine#TestRun_SuccessfulTickResetsErrorCount`
- `internal/orchengine#TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`
- `internal/orchengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D8, D9's orch half, and D11 for `internal/orchengine`.

D8: `logIdleStateDisagreement` treats two more combinations as agreement: an idle probe beside a `busy` state whose cause is `shuttleengine.SessionCauseBackground`, and a probe that is not idle beside `asking`, which leaves `asking` out of the not-idle disagreement set.
A remaining disagreement logs `orch: session state disagrees with the idle probe` at Info, once per episode:
`idleStatePair` gains the state's `Since`, the episode key is the idle answer, state, cause and `Since`, and the watcher logs only when the pair differs from the last pair it logged.
A probe that agrees no longer clears that memory, so a flip back to agreement and then to the same disagreement with the same `Since` does not log again, while a new `Since` or any other changed field does.
The doc comment names the agreement cases and the episode key, and so does `doc.go`'s session-state paragraph, which today lists `asking` as a disagreement, says the line logs at Warn and that it logs again after either side changes.

D9: `heldReason` answers `draftHoldReason` for a probe with `Draft` set, and `heldTyping` uses `draftHoldReason` as its reason for an error `shuttleengine.IsDraftHold` reports, in place of the error's first line.
So `orch: injection held`, `orch: injection held by the session` and `State.Stuck` carry the constant and never the operator's draft text, and the existing once-per-change dedupe holds while the operator types, since the reason no longer changes with each keystroke.
Every other probe reason and send error keeps its own text.

D11: `Watcher.Run`'s `orch: watcher tick failed` Warn fired on every failing tick up to `maxConsecutiveTickErrors` (30), past the policy's roughly-ten bound: a `logger.Episode` local to `Run` logs the first failure of a run of failures at Warn and the rest at Debug, each with the error and the `consecutive` count, and the next successful tick calls `Recover` with `orch: watcher tick succeeded again`; the cap, its exit record and its `record watcher exit failed` Warn are unchanged.
`orchNetworkOperationInFlight`'s `orch: the pause's network operation markers cannot be read` Warn, asked on every tick of a pause's grace while the orch reads idle, goes through a new `Watcher` field `netOpsFailures logger.Episode`, with `Recover` on the next successful read logging `orch: the pause's network operation markers are readable again`.

Tests:
`fakeSession` gains `draft bool`, reported as `IdleProbe.Draft` beside `reason` on every probe that is not idle.
`TestWatcher_LogsDisagreementBetweenIdleProbeAndSessionState` gains a `cause` column (default `turn`), counts the disagreement line and asserts each counted line is `level=INFO`, and changes its rows:
"a not-idle probe beside asking" now agrees and logs 0;
a new row "an idle probe beside busy on background work agrees" (cause `background`) logs 0;
"a disagreement warns again after an agreement between" becomes "a disagreement with the same since logs once across an agreement between" and wants 1;
and a `sinces` column, consumed one per tick into the scripted state's `Since` (the last one sticking), drives a new row "a disagreement with a new since logs again": an idle probe beside `busy` on three ticks whose `Since` is t0, t0, t1, wanting 2.
`TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`, whose rows already reach `orchNetworkOperationInFlight` through `watchEnv.orchMarker`, gains a row that writes a plain file at `pauserecord.NetDir(e.pausePath)`, so `ListNetOps` fails on every grace tick while the orch reads idle, ticks three times within the grace, and asserts exactly one `level=WARN` `network operation markers cannot be read` line;
it then replaces that plain file with an empty directory, ticks once more within the grace, and asserts one `level=INFO` `network operation markers are readable again` line.
That row captures the logger at Info, so it runs serially.
`TestWatcher_DraftHoldLogsOnceWithoutTheDraftText` is serial (it captures the process-global logger) with four rows, each driving three ticks whose draft grows by one character per tick (`a`, `ab`, `abc`) through a queued notice:
the probe route, with `fakeSession.idle` false, `draft` true and `reason` quoting the draft;
the send route held by the idle wait, with the probe idle and `sendErrs` three `*shuttleengine.SessionBusyError` values with `Draft` set whose messages quote the draft;
the send route refused over foreign box text, with `sendErrs` three errors wrapping `shuttleengine.ErrForeignBoxText` and quoting the draft;
the send route whose text merged with a draft, with `sendErrs` three errors wrapping `shuttleengine.ErrSentTextInDraft` and quoting the draft.
Each row asserts exactly one of its held lines (`orch: injection held` for the probe, `orch: injection held by the session` for the send), that no log line contains the draft text, and that `State.Stuck` is `draftHoldReason`.
`TestRun_SuccessfulTickResetsErrorCount` captures at Info and additionally asserts two `level=WARN` `watcher tick failed` lines, one per run of failures, and one `level=INFO` `watcher tick succeeded again` line between them.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/orchengine` row rises from 139 to 140.

**ImpactSummary:** watcher logging and the text stored in `State.Stuck` for a draft hold change in `internal/orchengine`; no injection, cycle or exit decision changes, and `orch status` shows the constant draft reason in place of the quoted draft.

**Commit:** 12: orch: agree on benign probe states, keep drafts out of holds, log tick failures once
