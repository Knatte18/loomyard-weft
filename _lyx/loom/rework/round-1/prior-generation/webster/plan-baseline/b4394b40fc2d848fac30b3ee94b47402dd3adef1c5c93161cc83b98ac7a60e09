# Card 4 — reapply-no-zoom-flicker

**Edit:**
- `internal/reedengine#Engine.reapplyLayout` -> `func (e *Engine) reapplyLayout(lastApplied, lastFailed render.Box, probeHook bool) (ReapplyResult, error)`
- `internal/reedengine#Engine.handleWatchOutcome` -> `func (e *Engine) handleWatchOutcome(mode watchMode, state *watchState, t watchTiming, res ReapplyResult, err error, lastApplied, lastFailed *render.Box, dormantFrom *watchMode) watchMode`
- `internal/reedengine#pollWakeChanged` -> `func pollWakeChanged(res ReapplyResult, err error, lastApplied, lastFailed render.Box) bool`
- `internal/reedengine#Engine.watchLoop`
- `internal/reedengine#TestReapplyLayout`
- `internal/reedengine#TestReapplyLayout_Deferral`
- `internal/reedengine#TestReapplyLayout_HookProbeExactMatchOnly`
- `internal/reedengine#watchPollWakeChangedPerOutcome`
- `internal/reedengine#TestWatchLoop_FailuresNeverKillTheLoop`
- `internal/reedengine#TestWatchLoop_PollModeByDefault`
- `internal/reedengine#waitForPromotion`

**Intent:** The watchdog's re-apply stops toggling zoom on a re-apply that changes nothing (D9).

`reapplyLayout` keeps its try-lock, session, state, window and pane reads and its hook probe, all outside the zoom bracket, then applies `applyLayoutLockedOpts`' two session-survival guards (fewer than two panes, no placed strand) itself and returns with nothing applied when either holds.
It then resolves the live box through `liveBoxLocked` and records it, live or not, in the result.
A live box equal to `lastApplied` or to `lastFailed` returns with `Applied` false and no bracket entered, so a zoomed window gets no `resize-pane -Z`.
Only an apply it will run enters `keepZoomLocked`, which re-lists the panes on the unzoomed window and calls `applyLayoutLockedOpts` with `SkipFocus` and no box guard.
A failed apply returns its error with the result still carrying the live box it tried.
`handleWatchOutcome` records that box in `lastFailed` when the failed result's box is live.
A result with `Applied` set records its live box in `lastApplied` as today and clears `lastFailed` back to the zero box, so a box that failed once is skipped only until another box applies.
A live result with nothing applied (a skip) records its box in `lastApplied` only when it differs from `lastFailed`, and never clears `lastFailed`, so a skip on the failed box leaves both boxes as they were.
`pollWakeChanged` counts a live box as a change only when it differs from both boxes, so a poll-mode watcher sitting on a failed box backs off instead of waking at the base cadence (`PATTERN-daemon-wakeups`).
`watchLoop` holds `lastFailed` beside `lastApplied` and passes both at every call; a strand change still re-applies through its own op, so a failed box is retried there or once the box changes.
Bound: the skip suppresses only a re-apply whose box an earlier watcher apply already answered, applied or failed, and never a strand op's apply.
The doc comments of `reapplyLayout`, `pollWakeChanged` and `handleWatchOutcome`, and `internal/reedengine/doc.go`'s watchdog paragraph, say so.

`TestReapplyLayout` passes a `lastFailed` per row (the zero box where a row sets none) and gains a `wantErr` field, with rows:
"EqualBoxOnAZoomedWindowTogglesNoZoom" (zoomed, live box equal to `lastApplied`, asserting no `resize-pane` call at all and no `select-layout`);
"BoxEqualToTheLastFailedBoxSkipsOnAZoomedWindow" (zoomed, live box equal to `lastFailed`, the same two assertions);
"AFailedApplyReportsItsLiveBox" (`select-layout` scripted to fail, wanting the error with `BoxIsLive` true and the queried box).
The existing "ZoomedWindowIsBracketedAroundTheApply" row keeps pinning the unzoom, apply, re-zoom order for a box that does apply, and the `//testtiming:keep` reason gains "an equal or failed box skipping the zoom bracket".
`TestReapplyLayout_Deferral` and `TestReapplyLayout_HookProbeExactMatchOnly` pass the zero `lastFailed` and change nothing else.
`watchPollWakeChangedPerOutcome` gains the row "LiveBoxEqualToTheLastFailedBox", wanting no change, and its doc comment adds the `lastFailed` rule.
An applying re-apply now lists the panes twice, once before the bracket for its guards and once inside it on the unzoomed window, while a skipped one lists them once: `TestWatchLoop_PollModeByDefault`'s `listPanesPerReapply` bound and its comment, and `waitForPromotion`'s list-panes counting and its comment, account for that.
`TestWatchLoop_FailuresNeverKillTheLoop` changes with the rule: after the first failed attempt at the 140x50 box, the retry streak at that box adds no further `select-layout`; a fresh signal with the box still 140x50 adds none either; a fresh signal after the box answer changes to 141x50 adds one attempt, so the loop is still running and responsive.
Its doc comment says so.

**ImpactSummary:** Three unexported watchdog members re-signed, all called only from `watchLoop` and the reapply and watch-loop tests this card lists; the strand ops' apply path is untouched.
