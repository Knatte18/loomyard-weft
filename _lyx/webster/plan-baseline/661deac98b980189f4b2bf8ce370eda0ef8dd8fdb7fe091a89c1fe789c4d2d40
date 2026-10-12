# Card 3 — reed reads own-window panes through a session listing and never as window 0

**Create:**
- `internal/reedengine#ownWindowOption`
- `internal/reedengine#groupWindowOption`
- `internal/reedengine#windowPane`
- `internal/reedengine#windowPane.livePane`
- `internal/reedengine#windowPaneFormat`
- `internal/reedengine#parseWindowPanes`
- `internal/reedengine#TmuxCmd.listWindowPanes`
- `internal/reedengine#isOwnWindow`
- `internal/reedengine#presentPanes`
- `internal/reedengine#Engine.listStrandAndWindowPanes`
- `internal/reedengine#Engine.selectStrandsWindowIfCurrentLocked`
- `internal/reedengine#windowPaneListingOf`
- `internal/reedengine#fakeTmux.answerWindowPanes`
- `internal/reedengine#TestStatus_ReportsOwnWindowStrandsFromTheSessionListing`

**Edit:**
- `internal/reedengine#recordedPaneIDs`
- `internal/reedengine#TmuxCmd.strandWindowTarget`
- `internal/reedengine#reconcilePlan`
- `internal/reedengine#planReconcile` -> `func planReconcile(strands []Strand, live []LivePane, windows []windowPane, policy reapPolicy) reconcilePlan`
- `internal/reedengine#Engine.reconcileLocked` -> `func (e *Engine) reconcileLocked(st *ReedState, live []LivePane, windows []windowPane) (killed []string, err error)`
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#Engine.reconcileApplyPersistLocked`
- `internal/reedengine#Engine.Resume`
- `internal/reedengine#StrandStatus`
- `internal/reedengine#Engine.Status`
- `internal/reedengine#Engine.resolvePaneInThisSessionLocked`
- `internal/reedengine#Engine.repairNames`
- `internal/reedengine#Engine.Directory`
- `internal/reedengine#Engine.AddStrandUnless`
- `internal/reedengine#anyPlacedStrand`
- `internal/reedengine#windowListFormat`
- `internal/reedengine#Engine.findWindowLocked`
- `internal/reedengine/doc.go`
- `internal/reedengine#fakeTmux`
- `internal/reedengine#fakeTmux.exec`
- `internal/reedengine#TestPlanReconcile`
- `internal/reedengine#TestReconcileLocked_NoDeadPanes_ClearsGoneBindingsWithoutTouchingTmux`
- `internal/reedengine#TestReconcileLocked_LogsTheUntrackedPanesItReaps`
- `internal/reedengine#TestStrandWindowTarget`
- `internal/reedengine#TestResolvePaneInThisSessionLocked`
- `internal/reedengine#TestRepairNames_SessionName`
- `internal/reedengine#TestAnyPlacedStrand`
- `internal/reedengine#TestOpenWindow`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#Engine.listStrandPanes`
- `internal/reedengine#Engine.strandWindowTargetFor`
- `internal/reedengine#aliveIDSet`
- `internal/reedengine#liveIDSet`
- `internal/reedengine#sessionListingOf`
- `internal/reedengine#newTestEngine`

**Intent:**
D6, and the reconcile half of D5: an own-window strand's pane is read through a session-scoped listing, its window is never read as the strands' window, its dead pane keeps its binding, and an own window no strand holds is ended.
No strand can be added `own-window` until card 4, so this card is exercised through hand-built state; every window 0 behavior stays as it is.

New file `internal/reedengine/ownwindow.go` (package `reedengine`), with a file comment saying it holds the own-window placement: the session-wide window listing, the presence of own-window panes, and (cards 4 to 6) their windows' creation, grouping and removal.
It declares:

- `ownWindowOption` (`@lyx_own`, the marker reed sets on every own window it opens) and `groupWindowOption` (`@lyx_group`, the group name a shared window carries, set by card 5).
  Unlike the display-only options, `@lyx_own` and `@lyx_group` are read by Go: the window end and the group lookup decide on them, and a window without `@lyx_own` is never ended.
- `windowPane`: one pane of the session-wide listing, with `WindowID`, `Own bool`, `Group string` (empty for `-`), `PaneID`, `Dead bool`, `PID int` and `Title` (the remaining text, which may hold spaces).
  `livePane()` maps it to a `LivePane` with `ID`, `Dead`, `PID` and `Title`; the geometry fields stay zero, since no layout reads an own-window pane.
- `windowPaneFormat` and `parseWindowPanes`, which splits each line into seven fields with the title last and skips a line it cannot parse.
- `TmuxCmd.listWindowPanes(session)`: `list-panes -s -t =<session> -F windowPaneFormat`, parsed; the exact-match target keeps sibling sessions out.
- `isOwnWindow(d)`: `d.Anchor == render.AnchorOwnWindow`.
- `presentPanes(strands, live, windows)`: `live` (window 0's panes), plus the `livePane()` of every listed pane bound to an own-window strand of `strands` that `live` does not already hold.
  It is the pane set every liveness read uses, so an own-window pane counts as present (alive or dead), and a pane the operator killed by hand reads absent, as a window 0 pane does today.
- `Engine.listStrandAndWindowPanes(st)`: `listStrandPanes(st)` and then `tmux.listWindowPanes(e.SessionName())`, returning both; either failure is returned.
- `Engine.selectStrandsWindowIfCurrentLocked(st, ending)`: reads the session's current window (`display-message -p -t =<session>: '#{window_id}'`) and, when it is one of `ending`, runs `select-window -t <strandWindowTargetFor(st)>`, so a viewer on a window about to end lands on window 0 and a viewer elsewhere is never moved (D5's bound).
  Every failure is logged at Warn and never fails the step.

The strands' window seam:

- `recordedPaneIDs` skips own-window strands, so `TmuxCmd.strandWindowTarget` resolves through Selvage, then a strand placed in window 0, then the current window, and an own-window pane never makes its window read as the strands' window, including when Selvage is dead or absent.
  `strandWindowTarget`'s doc comment says "any recorded strand pane placed in window 0".

Reconcile:

- `reconcilePlan` gains `ownWindowsToEnd []string`.
- `planReconcile(strands, live, windows, policy)`:
  - An own-window strand's binding is cleared only when no listed pane in `windows` carries its pane id; a present pane keeps it, alive or dead, and is never scheduled for any kill.
    Window 0 strands are judged against `live` exactly as today.
  - The untracked reap, the dead-pane kill and the keep-one-dead-pane survival rule stay computed over `live` alone, which holds window 0's panes only, so an alive own-window pane never stands in for the survival rule and is never reaped.
  - `ownWindowsToEnd` lists, in listing order and once each, every window whose panes carry `Own` and none of whose pane ids is bound to any strand of `strands`; a window without `@lyx_own` never qualifies.
- `Engine.reconcileLocked(st, live, windows)` runs `selectStrandsWindowIfCurrentLocked(st, plan.ownWindowsToEnd)`, then `kill-window -t <id>` for each, logged at Info as `reed: reconcile ended own windows no strand holds` with the window ids; a failed kill returns its error like a failed `kill-pane`.
  The windows ended are not added to `killed`, since no window 0 pane went.
- The three callers list both sets through `listStrandAndWindowPanes` and pass them on: `launchStrandLocked`, `reconcileApplyPersistLocked` and `Resume`; each re-lists after a kill as today.
  Since every op reconciles under the op lock after an add has persisted its binding (`AddStrandUnless` saves before its reconcile tail, and a launch's own reconcile runs before its window exists), a group window whose first strand is mid-add is never ended.

The liveness reads take `presentPanes(st.Strands, live, windows)`:

- `StrandStatus` gains `Present bool` (the strand's binding names a present pane, alive or dead) and `OwnWindow bool` (the strand is anchored `own-window`), both documented as what shuttle's stop path decides on (card 8).
  `Status` fills them, and `Live` from `aliveIDSet` of the present set.
- `resolvePaneInThisSessionLocked`, so `SendText`, `SendKey`, `PasteSubmit`, `CapturePane`, `OperatorActive` and `SetWaitMark` reach an own-window pane, while a pane of another session is still refused.
- `repairNames`: the title repair and the session-name repair run over the present set, so an own-window strand with a session id gets its drifted session name restored through the guarded paste-submit (D12's name repair).
- `Directory`'s rows.
- `AddStrandUnless`'s `--if-absent` classification.
- `Resume`'s `planResumeLaunches` call takes `aliveIDSet` of the present set, so a live own-window strand is never relaunched beside itself.

`anyPlacedStrand` also skips own-window strands, so window 0's apply guard, the resize hook's pins and the watchdog's resize loop never count one, even for a pane id that some listing reports in window 0.

`OpenWindow`'s lookup:

- `windowListFormat` becomes `#{window_id} #{pane_id} #{pane_dead} #{?#{@lyx_own},1,0} #{window_name}`, and `findWindowLocked` splits five fields and skips a window whose fourth reads `1`.
  `OpenWindow` therefore never adopts, kills or replaces an own window of the same name: it opens its own.

`internal/reedengine/doc.go`'s "Strand window" paragraph says own-window strands live outside the strands' window, are read through the session listing, and that only a window carrying `@lyx_own` with no bound pane is ever ended by reconcile; the "Detached windows" paragraph says `OpenWindow` skips own windows.

Test support in `faketmux_test.go`:

- `fakeTmux` gains `windowPanes *tmuxAnswer`, set by `answerWindowPanes`.
- `fakeTmux.exec`: a session-wide listing whose format argument is `windowPaneFormat` answers `windowPanes` when set, and otherwise `windowPaneListingOf` of the plain `list-panes` script.
- `windowPaneListingOf` renders each pane of a `list-panes` answer as one `windowPaneFormat` line in window `fakeStrandWindow`, not own and with no group, keeping its dead flag, pid and title.
  So every existing test whose fake scripts only `list-panes` sees the same panes, in window 0, through the new listing.

Tests:

- `TestPlanReconcile` gains a `windows []windowPane` row field and an `wantOwnWindowsToEnd []string` assertion, and rows:
  - `OwnWindowStrandPresentAliveKeepsBinding`: an own-window strand bound to `%5`, listed alive in window `@3`, absent from `live`: not cleared, nothing killed.
  - `OwnWindowStrandPresentDeadKeepsBindingAndIsNeverKilled`: the same with `%5` dead, beside an alive window 0 pane: not cleared, `%5` in neither kill list.
  - `OwnWindowStrandAbsentIsCleared`: `%5` in no listing: cleared.
  - `AliveOwnWindowPaneDoesNotSatisfyTheSurvivalRule`: every window 0 pane dead and an own-window pane alive: the first dead window 0 pane is still kept.
  - `OwnWindowWithNoBoundPaneIsEnded`: window `@4` listed `Own` with a pane no strand holds: `wantOwnWindowsToEnd: ["@4"]`.
  - `WindowWithoutTheMarkerIsNeverEnded`: the same pane in a window not `Own`: nothing ended.
  - `OwnWindowHoldingABoundPaneIsKept`: `@3` holds a bound pane and an unbound one: nothing ended.
- `TestReconcileLocked_NoDeadPanes_ClearsGoneBindingsWithoutTouchingTmux` and `TestReconcileLocked_LogsTheUntrackedPanesItReaps` pass `nil` windows, so neither issues a new tmux call.
- `TestStrandWindowTarget` gains `OwnWindowStrandPaneIsNotTheStrandsWindow`: no Selvage, one own-window strand bound to `%2` listed in `@5`: the target is the current window, and `SelvageWindowWinsOverAnOwnWindowStrand`: Selvage `%1` in `@4` and an own-window strand `%2` listed first in `@5`: `@4`.
- `TestResolvePaneInThisSessionLocked` gains a `strand` row field (the existing rows keep the window 0 strand) and `an own-window pane outside window 0 resolves`: an own-window strand bound to `%7`, `list-panes` answering `%1` only and `answerWindowPanes` listing `%7` in `@3`: resolves `%7`.
- `TestRepairNames_SessionName` gains row fields `ownWindow bool` and `windowPanes string` (zero for the existing rows).
  A row with `ownWindow` builds its strand with `Display.Anchor` `own-window` instead of `repairStrand`'s below-parent, re-scripts `list-panes` with no pane `%1` after `newRepairTestEngine`, and calls `answerWindowPanes(windowPanes)`.
  The new row `OwnWindowStrandRenamedFromTheSessionListing` lists `%1` alive in `@3` with the strand's title: the rename is typed.
- `TestAnyPlacedStrand` gains a row: an own-window strand whose pane id is in the present set is not placed.
- `TestOpenWindow`'s existing session-listing fixtures become five-field lines with the own flag `0` in fourth place (`"@0 %1 0 0 tmux\n@4 %9 0 0 "+name+"\n"` and `"@4 %9 1 0 "+name+"\n"`), so its live-window and dead-window rows keep their meaning.
  A new row lists `"@4 %9 0 1 "+name+"\n"`, a live own window of the requested name: it is neither returned as existing nor killed, and a new window is opened.
- `planResumeLaunches` reads no anchor but `hidden`, so `TestPlanResumeLaunches_ThreeLifecycleStates` already pins the hold and relaunch an own-window strand takes; Resume's new part, the present set, is pinned by card 4's Resume subtest.
- New `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing` in `lifecycle_test.go`: a window 0 strand on `%1` and own-window strands on `%5` (alive in `@3`), `%6` (dead in `@3`) and `%9` (in no listing).
  Status reports `%5` live, present and own-window; `%6` not live, present and own-window; `%9` neither live nor present; the window 0 strand live, present and not own-window.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/reedengine` by one for the new test.

**ImpactSummary:** Re-signs `planReconcile` (one caller and its test) and `reconcileLocked` (three callers and two tests); every liveness read gains one session listing call, which the fake answers from the existing `list-panes` script, so tests that script only window 0 stay unchanged.

**Commit:** 3: reed reads own-window panes from the session listing and ends orphan own windows
