# Card 3 — FollowRun: turn follow on and select the relevant window

**Create:**
- `internal/reedengine#Engine.FollowRun`
- `internal/reedengine#TestFollow_RealTmux`

**Edit:**
- `internal/reedengine#Engine.selectRelevantWindowLocked`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#followSessionOption`
- `internal/reedengine#Engine.followingLocked`
- `internal/reedengine#Engine.AttachArgv`

**Intent:**
D2's reed half: the one engine operation that turns follow on.
`FollowRun`, in `follow.go`, runs under `withOpLock`: `requireSessionLocked`, then `loadOrInitStateLocked` (read-only, no `SaveState`), then `set-option -t <exactSessionTarget(session)> @lyx_follow on`, then `selectRelevantWindowLocked(st, nil)`, which selects the newest live agent window, else the overview, else nothing.
It returns the no-session and state-load errors; a failed option write is logged at Warn and the select still runs, and a failed listing or select is logged by `selectRelevantWindowLocked` and never fails the call.
The bound: it sets one session option and selects one window of the engine's own session; no other reed operation but this one and card 5's verb turns follow on, and `AddStrand`, `ReplaceStrand`, `Resume` and `Up` never call it, so the orch's `runs` overview, which goes through `AddStrand`, never turns follow on.
`selectRelevantWindowLocked` now also serves a viewer anywhere: its doc comment drops "for a viewer already known to be on one of them" for "for a viewer on one of ending, or for any viewer when ending is empty", and its two Warn texts drop "a viewer may be left on a window that ends" for "the viewer is left where it is", which holds for both callers.

`TestFollow_RealTmux` is a new real-tmux test in a new file `internal/reedengine/follow_integration_test.go` under `//go:build tmux && linux`, attaching clients through `startInPTY` from `attachgeometry_integration_test.go` with `AttachArgv`'s argv, and reading each client's window with `display-message -p -c <client> "#{window_id}"`.
The test attaches four clients in all, none detached before the session is killed: the first attach waits with `waitForClientAttached`; the second, third and fourth wait through `waitUntil` until `list-clients -t =<session>` reports 2, 3 and 4 clients, and each client is named from that listing's `#{client_name}` format before its `window_id` is read.
Every client of the session shares its current window, so "every client on the shell" is reached by one `select-window -t <shell window>`, and an assertion about a client's window is made for every attached client.
Its subtests run in order on one session:
- `BootSetsFollowOn`: after `Up`, `@lyx_follow` reads `on`.
- `RunStartSelectsTheOverviewThenEachNewAgentForEveryClient`: the first and second clients attach, waiting for count 1 and 2, while the session holds no overview and no agent, both on the shell; an overview strand added with `AddStrand` leaves both on the shell and the follow state unchanged; `FollowRun` moves both to the overview window; each of two own-window strands added after it moves both to its window in turn.
- `FollowRunTurnsAFollowOffSessionOnAndSelectsTheRelevantWindow`: with `@lyx_follow` set off and the viewer on the shell, `FollowRun` reads back `on` and selects the newest live agent window; with every agent removed it selects the overview; with the overview removed too it selects nothing and leaves the viewer on the shell.
- `ASecondAttachMovesTheFirstClientOnlyWhileFollowIsOn`: with an overview present, follow on and every attached client put on the shell, the third client's attach (count 3) lands all three on the overview; with follow off and every attached client put back on the shell, the fourth client's attach (count 4) leaves all four on the shell; `@lyx_follow` reads after each attach what it read before.
- `AResumeOfALiveSessionChangesNoFollowState`: with follow off, the overview strand live and the viewer on the shell, `Resume` leaves `@lyx_follow` off and the viewer on the shell.
- `AFailedFollowWriteStillSelects`: a fake installed with `installFakeTmux` and `forwardTo` the real `TmuxCmd` fails only the `set-option` carrying `@lyx_follow`, through `answerFunc`; with the viewer on the shell, `FollowRun` returns nil, logs the Warn (captured with `logcapture.CaptureVerbose`) and still selects the relevant window.
- `NoSessionIsAnError`: run last, after `kill-session` of the engine's session, `FollowRun` returns the no-session error and issues no `set-option`.

D6 with follow on and off and the overview add and replace outside `FollowRun` are pinned by card 1's readbacks in the existing window-order and own-window subtests, so this test adds no subtest for them.
The state-load error is the one `loadOrInitStateLocked` returns to every engine operation and needs no row of its own here.

`test-budget.yaml`'s `internal/reedengine` row rises by one for `TestFollow_RealTmux`.
`doc.go`'s focus-rules paragraph names `FollowRun` as the one operation that turns follow on, alongside card 5's verb.

**ImpactSummary:** Adds one exported method and its test; no existing caller or test changes, and the budget row moves by one.

**Commit:** 3: reed: FollowRun turns follow on and selects the relevant window
