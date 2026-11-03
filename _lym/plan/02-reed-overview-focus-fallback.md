# Card 2 — reed-overview-focus-fallback

**Edit:**
- `internal/reedengine#agentWindowPanes` -> `func agentWindowPanes(st *ReedState, panes []windowPane, skip map[string]bool) (live, bound, overview map[string]bool)`
- `internal/reedengine#relevantWindow`
- `internal/reedengine#followTargetWindow`
- `internal/reedengine#Engine.FollowRun`
- `internal/reedengine#Engine.selectRelevantWindowIfCurrentLocked`
- `internal/reedengine#Engine.reconcileLocked`
- `internal/reedengine#Engine.killStrandPanes`
- `internal/reedengine/doc.go`
- `internal/reedengine#relevantWindowFixture`
- `internal/reedengine#TestAttachArgv_SelectsTheRelevantWindow`
- `internal/reedengine#TestFollowSessionVia`
- `internal/reedengine#TestWindowOrder_RealTmux`

**Uses:**
- `internal/reedengine#isOverview`
- `internal/reedengine#windowOfPane`

**Intent:**
D2: every focus rule that falls back to the overview window takes the lowest-index one.

- `agentWindowPanes` returns `overview` as the set of windows holding a pane bound to an overview strand, skip applied, instead of the one window of the last such pane.
- `relevantWindow` keeps the newest live agent window first; with none it returns the lowest-index listed window of that set, else the empty string.
  That carries the rule to attach, a viewed window ending (`killStrandPanes` and reconcile through `selectRelevantWindowLocked`, which passes the ending windows as skip, so a viewer on an ending overview window lands on another overview window) and `FollowRun`.
  No overview window is an agent window, so a live agent still wins over every overview window.
- `followTargetWindow` keeps the newest live agent rule and falls back to the lowest-index window carrying the overview marker, tracking that window's index instead of keeping the last one listed.
  An overview window an older lym opened carries no marker and still counts as an agent window, as its doc comment says.
- `doc.go`: the FollowSession paragraph and the focus-rule paragraph ("relevantWindow is the target of two of them") say "else the lowest-index overview window"; the `FollowRun` sentence likewise.
- The doc comments of the rule's callers say the same in place of "the overview": `FollowRun` ("else the overview, else nothing"), `selectRelevantWindowIfCurrentLocked` ("the newest live agent or the overview"), `reconcileLocked` ("the newest live agent, else the overview") and `killStrandPanes` ("the newest live agent, else the overview"); only the comments change.

A viewer ends up on an ending overview window through its strand's removal or replacement.
A pane that exits stays as a dead pane under remain-on-exit, and reconcile keeps a present own-window pane's binding, alive or dead, and never kills it (`internal/reedengine/reconcile.go`'s `planReconcile`), while it ends only own windows no strand holds; so an overview window ends only at its strand's removal or a relaunch, which the removal row below reaches.

Tests:
- `relevantWindowFixture` (tier0 helper) gains a second overview strand `runs` bound to `%13` in window `@4` at index 0, with `status`'s `@3` moved to index 1 and the agents and shell shifted up one; the pane listing lists `@3`'s pane after `@4`'s, so the old last-listed rule would answer `@3`.
- `TestAttachArgv_SelectsTheRelevantWindow`: the row `NoLiveAgentFallsBackToTheOverview` becomes `NoLiveAgentFallsBackToTheLowestIndexOverview` with `wantWindow: "@4"`; the live-agent rows keep `@2` and `@1`, so a live agent wins over both overview windows.
- `TestFollowSessionVia` (tier0) gains `NoLiveAgentSelectsTheLowestIndexOverviewOfTwo`: listing `@0 5 0 0 0\n@3 0 1 1 0\n@5 1 1 1 0\n` selects `@3`; the lower-index overview is listed first, so the old last-listed rule would answer `@5` and fail the row.
  A live agent winning over overview windows is already pinned by `NewestLiveAgentWinsOverOlderAgentDeadNewerAgentOverviewAndNonOwnWindow`, so no row is added for it.
- `TestWindowOrder_RealTmux` (tier2) gains `WithNoLiveAgentEndingTheViewedLowestOverviewSelectsTheNextLowest`, appended after card 1's last subtest `EqualKeysSitInTheOrderTheyLastOpened`, where the overview windows read `beam: prime, third, pawl`: remove the agent card 1 added, select `beam: prime`'s window, remove `beam: prime`, want the current window to be `third`'s, never `pawl`'s or the shell.
  Two overview strands remain in state, `third` at the lower index and `pawl` listed after it, so the old last-listed fallback would answer `pawl` and fail the row.

**ImpactSummary:** reed's focus fallback only: `agentWindowPanes` has the one caller `relevantWindow`, whose callers (attach, a viewed window ending, `FollowRun`) keep their signatures; `followTargetWindow`'s one caller `followSessionVia` is unchanged.

**Commit:** 2: reed: fall back to the lowest-index overview window
