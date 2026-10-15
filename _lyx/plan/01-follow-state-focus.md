# Card 1 — Follow state and the new-agent-window rule

**Create:**
- `plan:internal/reedengine#followSessionOption` -> `const followSessionOption = "@lyx_follow"`
- `plan:internal/reedengine#Engine.followingLocked` -> `func (e *Engine) followingLocked() bool`

**Edit:**
- `internal/reedengine#newAgentWindowSelected` -> `func newAgentWindowSelected(display render.Display, following bool) bool`
- `internal/reedengine#Engine.newAgentWindowTakesViewerLocked` -> `func (e *Engine) newAgentWindowTakesViewerLocked(display render.Display) bool`
- `internal/reedengine#Engine.openOwnWindowLocked`
- `internal/reedengine#Engine.serverSpawnArgv`
- `internal/reedengine#TestNewAgentWindowSelected`
- `internal/reedengine#TestServerSpawnArgv`
- `internal/reedengine#TestWindowOrder_RealTmux`
- `internal/reedengine#TestOwnWindow_RealTmux`
- `internal/reedengine#TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks`
- `internal/reedengine/doc.go`

**Delete:**
- `internal/reedengine#driverWindows`

**Uses:**
- `internal/reedengine#newAttachTestEngine`
- `internal/reedengine#relevantWindow`

**Intent:**
D1 and D3: the per-session follow state and the new-agent-window rule built on it.

Add a new file `internal/reedengine/follow.go` holding `followSessionOption`, the session user option `@lyx_follow`, and `followingLocked`.
`followingLocked` reads the option with `display-message -p -t <exactSessionWindowTarget(session)> "#{@lyx_follow}"` through `e.tmux` and reports false only for the trimmed value `off`; any other value, the empty one of an unset option included, reads on, so a session booted by an older lyx follows.
A failed read reports false and logs at Warn, naming socket, session and error, so a broken read never moves a viewer.
The file's header comment says the option is the one decision-bearing session option reed reads, unlike `@strand`, `@strand_color`, `@lyx_unseen` and the wait marks, which stay display only.

`serverSpawnArgv` chains `";", "set-option", "-t", target, followSessionOption, "on"` after the `@lyx_reed_session` tag off Windows, so boot sets follow on explicitly; on Windows the chain stays absent and the unset option reads on.

`newAgentWindowSelected` becomes pure over the display and the follow state: `display.Focus` returns true, else `display.KeepViewer` returns false, else it returns `following`.
The viewer-position branches (overview, all-dead agent window, relevant window, driver window) go, and with them `driverWindows`, whose only caller is `newAgentWindowSelected`, and the `agentname` import of `ownwindow.go` if nothing else there uses it.
`newAgentWindowTakesViewerLocked` drops its state parameter and its window and pane listings: a Focus or KeepViewer display decides with no read, and any other reads `followingLocked`.
Its one caller, `openOwnWindowLocked`, passes only the display; the window it opens never changes the follow state, and the selected window still carries no `@lyx_unseen`.
`relevantWindow` stays as it is, for attach and the ending-window moves (D6).

The bound this card introduces: with follow on, a new agent window now takes a viewer on the shell or on an older agent window, which the doc forbade; the follow state is the bound, and a `Focus` strand still takes the viewer whatever it reads.

Tests:
`TestNewAgentWindowSelected` is rewritten to drive `newAgentWindowTakesViewerLocked` on an engine from `newAttachTestEngine` whose fake answers the format `#{@lyx_follow}` through `answerFormat`.
Its rows: a plain own-window display with the read answering empty (unset, older lyx), `on` and `off` gives true, true and false; a read that errors gives false and logs the Warn; a Focus display with the read answering `off` gives true; a KeepViewer display with the read answering `on` gives false; Focus beside KeepViewer gives true; and the Focus and KeepViewer rows issue no `display-message` read of the format.
`TestServerSpawnArgv`'s non-Windows want gains the `@lyx_follow on` set-option; its Windows row stays without it.
`TestWindowOrder_RealTmux` sets `@lyx_follow` off on its session right after `Up` and again right after the `Resume` in `ABelowParentStrandRelaunchedByResumeIsBackInTheShellWindow`, since a restarted server boots it on, so the window-order subtests keep their viewer.
Its four new-agent-window subtests (`ANewAgentWindowIsSelectedFromTheOverviewAndFromAnAllDeadAgentWindow`, `ANewAgentWindowIsSelectedFromTheNewestLiveAgentAndFromTheDriver`, `ANewAgentWindowKeepsAViewerOnAnOlderLiveAgentOrTheShellAndCarriesUnseen`, `AFocusStrandIsSelectedFromTheShell`) are rewritten around the follow state: with follow on, a new agent window is selected from the shell and from an older live agent window and carries no `@lyx_unseen`; with follow off, it leaves the viewer on the shell, on an older live agent window and on the overview, and carries `@lyx_unseen`; a Focus strand is selected from the shell with follow off; a KeepViewer strand leaves a viewer on the shell with follow on; and no add changes `@lyx_follow`. Each subtest leaves follow off when it ends.
`TestWindowOrder_RealTmux`'s `ReplacingTheViewedAgentLandsTheViewerOnTheReplacement` sets follow on before its replace, since with follow off the replacement's new agent window no longer takes the viewer from the older agent it fell back to, and sets it off again when it ends.
D6 and the overview-outside-the-bootstrap criterion are pinned by readbacks in the existing subtests that already drive those moves, each asserting `@lyx_follow` reads after the step what it read before:
`RemovingTheViewedAgentWithNoLiveAgentLeftSelectsTheOverview` and `ReplacingTheViewedOverviewSelectsTheNewOverviewWindow` with follow off, as the test leaves it, `AnOverviewAddedAfterAgentWindowsHoldsTheLowestIndexAndCarriesNoMarks` with follow off and the viewer left where it was, and `TestOwnWindow_RealTmux`'s `RemoveMovesAViewerOnTheEndingWindowToTheNewestLiveAgent` with follow set on for that subtest and off again when it ends, so the ending-window move is pinned with follow on and off.
`TestOwnWindow_RealTmux` sets `@lyx_follow` off right after `Up`, so `OpensDetachedNamedByRoleWithoutMovingTheViewer` keeps its assertions; `ResumeAfterAServerRestartReopensDetached` asserts that the restarted session reads follow on and that the relaunched own window took the viewer, then sets follow off again.
`TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks` sets `@lyx_follow` off before it adds `scout`, so the new window still leaves the viewer on `extra` and carries `@lyx_unseen` as its assertions expect.

`doc.go`'s focus-rules paragraph is rewritten: the follow state, its default and its read-failure rule, the three automatic switch points with the new-agent rule, and D6's ending-window moves, which neither read nor change the follow state; the sentence saying no switch moves a viewer off the shell goes.
The display-only sentence of the pane-options paragraph (`doc.go` beside `@strand` and `@strand_color`) gains that `@lyx_follow` is the one decision-bearing session option reed reads, those options and the wait marks staying display only.

**ImpactSummary:** `newAgentWindowSelected` and `newAgentWindowTakesViewerLocked` each have one caller, both on this card; `driverWindows` has no other caller; the boot argv change is pinned by `TestServerSpawnArgv`, and the two real-tmux tests are the only ones asserting a viewer across own-window adds.

**Commit:** 1: reed: per-session follow state decides whether a new agent window takes the viewer
