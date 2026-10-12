# Card 6 — removing an own-window strand moves its viewer to window 0 and relays out a shared window

**Create:**
- `internal/reedengine#Engine.relayoutSharedWindowsLocked`

**Edit:**
- `internal/reedengine#Removed`
- `internal/reedengine#Engine.killStrandPanes` -> `func (e *Engine) killStrandPanes(st *ReedState, paneIDs []string) (reapPIDs []int, leftWindows []string)`
- `internal/reedengine#Engine.RemoveStrand`
- `internal/reedengine#Engine.ReplaceStrand`
- `internal/reedcli#reedCLI.removeCmd`
- `internal/reedengine/doc.go`
- `internal/reedengine#TestOwnWindow_RealTmux`

**Uses:**
- `internal/reedengine#Engine.selectStrandsWindowIfCurrentLocked`
- `internal/reedengine#TmuxCmd.listWindowPanes`
- `internal/reedengine#windowSplitLayout`
- `internal/reedengine#validateWindowConfig`
- `internal/reedengine#windowPane`

**Intent:**
D5's removal path and D3's remove rule: before an own window loses its last pane, a viewer on it is moved to the strands' window; a shared window a strand leaves is laid out again; and an invalid `window_split` or `window_share_max` never blocks a removal.

`killStrandPanes(st, paneIDs)`:

- It keeps card 4's session listing, membership filter and reap snapshot.
- Before any kill it groups the panes to kill by window from the listing; every own window (a window whose panes carry `Own`) all of whose listed panes are in the kill set is about to end, and `selectStrandsWindowIfCurrentLocked(st, ending)` runs first, so the client select always precedes the window's end and a viewer on any other window is never moved.
- It returns the reap pids as before, and in `leftWindows` the own windows that lose a pane but keep at least one.
- Its doc comment names the select and the returned windows.

`relayoutSharedWindowsLocked(windows)`:

- With `validateWindowConfig(e.cfg)` failing, it skips every layout and returns that error's text as the warning, so the removal itself still completes and only the even layout of the window it leaves is skipped.
- Otherwise it runs `select-layout -t <window> <layout>` for each, with the layout from `windowSplitLayout(e.cfg.WindowSplit)`; a failure is logged at Warn, and it returns "".
- A window to itself has no layout to redo: `leftWindows` names only windows that still hold a pane.

`Removed` gains `ConfigWarning string`: the invalid key a removal skipped the shared-window layout for, empty normally.

`RemoveStrand` calls `killStrandPanes(st, paneIDs)`, then, before its reconcile tail, `relayoutSharedWindowsLocked(leftWindows)`, and puts the returned text into the result's `ConfigWarning`.
Nothing on its path calls `validateWindowConfig` as a refusal, so shuttle's cleanup, a seat stop and D12's self-removal still remove under a `reed.yaml` typo.
`ReplaceStrand` takes the same two calls; its result has no warning field, so a non-empty warning is logged at Warn.

`reedCLI.removeCmd` adds `"warning": <text>` to its ok envelope when the result's `ConfigWarning` is non-empty.

`internal/reedengine/doc.go`'s "Own windows" paragraph gains the removal: the viewer moves to the strands' window before its window ends, a shared window is laid out again after a leave, and an invalid window key is reported, never refused, on removal.

`TestOwnWindow_RealTmux` gains subtests:

- `RemoveMovesAViewerOnTheEndingWindowToWindowZero`: `select-window` to an own window makes it the session's current window; `RemoveStrand` of its only strand leaves the strands' window current.
- `RemoveFromAnotherWindowLeavesTheViewer`: with own window A current, removing the strand of own window B leaves A current.
- `LeavingASharedWindowRelaysItOut`: removing one strand of a shared group of two leaves its window with one pane at the window's full width.
- `InvalidConfigStillRemovesAndReportsTheKey`: with `e.cfg.WindowSplit = "diagonal"`, `RemoveStrand` of a strand of a shared group returns no error, its pane is gone, `ConfigWarning` contains `window_split`; with `e.cfg.WindowShareMax = -1`, removing the last strand of an own window ends the window and the warning contains `window_share_max`.
- `SessionTeardownEndsTheShellBeforeTheChainRuns`, last since it tears the session down: an own-window strand whose `Cmd` runs `sleep 300` and then touches a marker; after `Down` returns, the marker does not exist, as for a pair teardown, which ends the reed session before any worktree goes.

**ImpactSummary:** Re-signs `killStrandPanes`, whose two callers, `RemoveStrand` and `ReplaceStrand`, are on this card; `Removed` gains one field the reed CLI prints; window 0 removals behave as before.

**Commit:** 6: Removing an own-window strand moves its viewer to window 0 and relays out its group
