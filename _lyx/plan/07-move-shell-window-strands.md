# Card 7 — Move a strand pane left in the shell window into its own window

**Create:**
- `internal/reedengine#planShellWindowMoves`
- `internal/reedengine#Engine.moveShellWindowStrandsLocked`
- `internal/reedengine#Engine.listReconcilePanesLocked`
- `internal/reedengine#Engine.markOwnWindowLocked`
- `internal/reedengine#strandsMarkerFormat`
- `internal/reedengine#markedStrandsWindow`
- `internal/reedengine#TmuxCmd.shellWindow`
- `internal/reedengine#TestListReconcilePanesLocked_MovesShellWindowStrands`

**Edit:**
- `internal/reedengine#TmuxCmd.strandWindowTarget`
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#Engine.reconcileApplyPersistLocked`
- `internal/reedengine#Engine.Resume`
- `internal/reedengine#Engine.openOwnWindowLocked`
- `internal/reedengine#TestStrandWindowTarget`
- `internal/reedengine#TestOwnWindow_RealTmux`

**Uses:**
- `internal/reedengine#Engine.listStrandAndWindowPanes`
- `internal/reedengine#Engine.ensureSelvagePaneLocked`
- `internal/reedengine#Engine.killReplacedOwnWindowPaneLocked`
- `internal/reedengine#strandLabel`
- `internal/reedengine#installFakeTmux`
- `internal/reedengine#newTestEngine`

**Intent:** D3's upgrade of live panes: a strand pane an older or `.dev-bin` binary left in the shell window is broken out into an agent window of its own, keeping its process, as the first step of every reconcile.

Shell-window resolution: `shellWindow` resolves the recorded Selvage pane's window first; when no recorded Selvage pane is listed it lists the session's windows with `strandsMarkerFormat` and resolves the window carrying `@lyx_strands`, parsed by `markedStrandsWindow` (the first line whose second field is `1`); it reports false when neither resolves.
`strandWindowTarget` returns `shellWindow`'s answer, and falls back to the session's current window only when it reports false.
That fallback stays for one case D3's sentence does not reach: a session with no recorded Selvage and no marked window, which is a fresh boot before its first Selvage split and marker, holding one window; the heal there splits that window, as today.
Since card 5 left `recordedPaneIDs` holding only Selvage, a viewer on an agent window can no longer make a heal resolve that window while a marker exists.
A failed window listing is returned as an error, as a failed pane listing is.

`planShellWindowMoves` is pure: it returns, in listing order, each pane of `shellPanes` that a strand of `st` is bound to, alive or dead, never the recorded Selvage pane and never an unbound pane, which the untracked reap still governs.
It never returns every pane of the window: when Selvage is not among `shellPanes`, the last bound pane stays, since `break-pane` on a window's only pane takes the whole window with its markers.

`listReconcilePanesLocked` resolves the shell window through `shellWindow`, lists its panes and the session's window panes, and returns them when `planShellWindowMoves` finds nothing.
When `shellWindow` reports false it plans no move at all and lists the current-window fallback as `listStrandAndWindowPanes` does, so a viewer on a shared group window can never have that window's bound panes broken out of it: the move only ever reads a window that the Selvage pane or the `@lyx_strands` marker names.
A failed listing, at the first listing or at the re-list after a move, is returned as an error, as `listStrandAndWindowPanes`' is.
Otherwise it calls `moveShellWindowStrandsLocked`, and when that moved a pane it lists both again, so reconcile judges a moved pane only as an own-window pane.
`moveShellWindowStrandsLocked` runs, per pane, `break-pane -d -b -s <pane> -t <shellWindow> -n <strandLabel(name)> -P -F #{window_id}`, which places the new window just before the shell window, behind every agent window, as `ownWindowInsertArgs` places a new one.
It then marks the window through `markOwnWindowLocked` (`@lyx_own`, the strand's `@strand_color`, `@lyx_unseen` off Windows) and sets the strand's `Display.Anchor` to `own-window` with no window group, which the op's own save tail persists; the pane's `@strand` options travel with the pane.
A failed `break-pane`, or an answer that is not one window id, is logged at Warn and leaves that pane and its anchor as they were for the next op, and the panes after it are still tried; a failed marker is logged like `openOwnWindowLocked`'s and the anchor is still rewritten, since the pane already sits in its own window.
A window whose `@lyx_own` failed to set carries the limit `openOwnWindowLocked`'s windows already carry: reconcile never ends it and `killReplacedOwnWindowPaneLocked` does not kill its old pane on a relaunch.
It reports whether any pane moved and logs each move at Info.
`openOwnWindowLocked`'s option block moves into `markOwnWindowLocked`, unchanged in effect.

`launchStrandLocked`, `reconcileApplyPersistLocked` and `Resume`'s head reconcile list through `listReconcilePanesLocked` instead of `listStrandAndWindowPanes`, so the move runs before the dead-pane kill, the untracked reap and the binding clears in every public op that reconciles.
`Up` and `Resume` heal Selvage before that tail, so the shell window holds Selvage or its corpse whenever a pane is broken out of it.
The move is keyed only on a bound strand pane in the shell window and gated by no upgrade marker, so a strand an older binary adds there later is moved by the next op.
A moved dead pane reaches reconcile as a present own-window pane and keeps its binding, so `Resume` relaunches it with `-a` after its window and `killReplacedOwnWindowPaneLocked` ends the old pane, leaving no second copy.

Tests: one fake-tmux test drives the op surface the three call sites share, `TestListReconcilePanesLocked_MovesShellWindowStrands`; `answerFunc` answers `list-panes` with the pre-move listing until a `break-pane` succeeds and with the post-move listing after, and each row asserts the `break-pane` argvs issued, the returned panes and windows, the strands' anchors and the list call count.
Rows: Selvage plus two bound strand panes moves both, re-lists, returns the post-move listing and rewrites both anchors to `own-window`; Selvage plus an unbound pane moves nothing and issues exactly `listStrandAndWindowPanes`' calls; a bound dead pane beside Selvage is moved; a sole bound pane with no Selvage, in the marked window, is not moved; two bound panes with no Selvage move the first only; the Selvage pane is never moved; no Selvage and no marked window, with the viewer's current window holding two bound own-window panes, issues no `break-pane` and no marker listing beyond the fallback's;
the first of two `break-pane`s failing and the second succeeding moves only the second, re-lists and leaves the first strand's anchor unchanged; every `break-pane` failing issues no re-list and changes no anchor; a malformed `break-pane` answer counts as a failure; a failed `@lyx_own` set still rewrites the anchor and logs at Warn; a failed first listing and a failed re-list each return the error.
`TestStrandWindowTarget` gains rows: no Selvage recorded with a marked window resolves it; a recorded Selvage that is not listed with a marked window resolves the marked one; no marked window falls back to the current window; a failed window listing is an error.
Its harness gains an answer and a call count for the marker listing beside `sessionListVerb`'s, and the rows that now reach the marker listing, `NothingRecordedIsTheCurrentWindowWithoutARoundTrip`, `NilStateIsTheCurrentWindow` and `OwnWindowStrandPaneIsNotTheStrandsWindow`, are renamed to say they read the marker and expect its one listing call, a nil state still resolving the current window with none.
At the tmux tier `TestOwnWindow_RealTmux` gains subtests that split a pane into the shell window and bind a strand to it through a `reed.json` written with `anchor: below-parent`:
a live pane survives the next op with its `#{pane_pid}` unchanged, in a window carrying `@lyx_own` placed before the shell window, its record reading `own-window`, and the shell window then lists exactly the Selvage pane, whose height equals the window's;
a dead one is moved the same way, and a `Resume` relaunches it into that window with no second copy of the pane left;
a `RemoveStrand` of a moved strand leaves the shell window and a viewer on it in place;
and a leftover pane that is the shell window's only pane, with Selvage killed, stays there through an `AddStrand` and a `RemoveStrand` with the shell window and its markers intact, after which an `Up` run while an agent window is selected heals Selvage in the shell window beside it and then moves it into its own window.

**ImpactSummary:** three reconcile call sites in `internal/reedengine` gain the move ahead of planning, and the shell-window seam gains a marker fallback; no exported signature changes and no other package is touched.

**Commit:** 7: reed: move strand panes left in the shell window into their own windows
