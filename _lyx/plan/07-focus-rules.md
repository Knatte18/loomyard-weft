# Card 7 — focus-rules

**Create:**
- `plan:internal/reedengine#relevantWindow` -> `func relevantWindow(st *ReedState, windows []sessionWindow, panes []windowPane, ending map[string]bool) string`
- `plan:internal/reedengine#newAgentWindowSelected` -> `func newAgentWindowSelected(st *ReedState, panes []windowPane, current string, focus bool) bool`
- `plan:internal/reedengine#Engine.selectRelevantWindowIfCurrentLocked` -> `func (e *Engine) selectRelevantWindowIfCurrentLocked(st *ReedState, ending []string)`
- `plan:internal/reedengine#Engine.attachWindowSelectionLocked` -> `func (e *Engine) attachWindowSelectionLocked(st *ReedState) []string`
- `plan:internal/reedengine#TestAttachArgv_SelectsTheRelevantWindow` -> `func TestAttachArgv_SelectsTheRelevantWindow(t *testing.T)`

**Delete:**
- `internal/reedengine#Engine.selectStrandsWindowIfCurrentLocked`

**Edit:**
- `internal/reedengine#Engine.killStrandPanes`
- `internal/reedengine#Engine.reconcileLocked`
- `internal/reedengine#Engine.openOwnWindowLocked`
- `internal/reedengine#Engine.AttachArgv`
- `internal/reedengine#Engine.warnMismatchedClientsLocked` -> `func (e *Engine) warnMismatchedClientsLocked(cols, rows int) (anotherClient bool)`
- `plan:internal/reedengine#TestWindowOrder_RealTmux`
- `internal/reedengine#TestOwnWindow_RealTmux`
- `internal/reedengine#TestAttachArgv_MultiClientWarning`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Apply D5's three focus points: attach, a new agent window, and a viewed window ending, each deciding from reed state, window ids and pane liveness only, never `@lyx_unseen`, `@lyx_hidden` read as a decision or a wait mark (`PATTERN-wait-mark-display-only`).

An agent window is a window holding a pane bound to an own-window strand; the overview, the shell and hidden windows are not agent windows.
`relevantWindow` returns, among the listed windows not in `ending`, the highest-index agent window holding a pane that is not dead, else the overview strand's window, else the empty string; by D2 the highest index is the newest live agent (on psmux, where the order is best-effort, it may be an older one).
`newAgentWindowSelected` reports whether a new agent window takes the viewer: always for a `Focus: true` strand, and otherwise only when `current` is the overview window or an agent window whose every pane is dead; the shell, a live agent window and any other window keep the viewer.

A viewed window ending: `selectRelevantWindowIfCurrentLocked` replaces `selectStrandsWindowIfCurrentLocked`, which is deleted.
When the session's current window is among `ending`, it lists the windows and panes and selects `relevantWindow` among those left, never the shell; with none left it selects nothing and leaves the choice to tmux.
A viewer elsewhere is never moved, and a strand leaving a shared window that stays open ends no window and switches nothing.
`killStrandPanes` (when `selectOnEnd` is set) and `reconcileLocked` (before it ends own windows no strand holds) call it; card 6's overview replace rule still takes precedence through `selectOnEnd`.

A new agent window: `openOwnWindowLocked`, for an own-window strand (never the overview), reads the session's current window before it opens the new one and, when `newAgentWindowSelected` holds, selects the new window and sets no `@lyx_unseen`; otherwise it sets `@lyx_unseen` as today.
A failed read of the current window keeps the viewer and logs at Warn.
This is where the orchestrator's `Focus: true` (card 9) selects its window once opened.

Attach: `attachWindowSelectionLocked` lists the windows and panes and returns `";", "select-window", "-t", <relevant window>`, or nothing when `relevantWindow` finds none or the listing fails (logged at Warn).
`AttachArgv` resolves it under the op lock right after card 5's shell check, ahead of every layout guard and also when no client size is known, and appends it to every argv it returns: the chained argv after its zoom restore, and the bare argv of every degrade path past the state read.
`warnMismatchedClientsLocked` now runs before that resolution on every path, warns about a size mismatch only when a client size is known, and reports whether another client is attached; a failed client listing counts as one, logged at Warn.
When another client is attached, `AttachArgv` appends no selection, so a second attach never moves a viewer.
Only a failure to take the lock, read the session or state, or list the attached clients drops the selection, each logged at Warn.
The chain's skip for a strands' window with fewer than two panes or no placed strand logs at Debug, since D3 and D4 make Selvage alone there the normal state; the other suppressions keep Warn.
Bound: an automatic switch happens only at these three points, never moves a viewer off the shell or a live agent window, and races an operator's own window change by at most one switch.
`AttachArgv`'s, `killStrandPanes`' and `reconcileLocked`'s doc comments and `internal/reedengine/doc.go` say so.
In `doc.go`, the Own windows sentence ending `by new-window -d, so it never takes focus` is rewritten to say a new agent window is selected under the focus rule, and the sentence `Removing an own-window strand moves a viewer on its window to the strands' window before the window ends` is rewritten to say the viewer goes to the relevant window, never the shell.
Three more `doc.go` sentences are rewritten.
The Own windows sentence "The window carries @lyx_own, @strand_color and @lyx_unseen, each set by window id" says an overview window carries neither `@strand_color` nor `@lyx_unseen`, and a new agent window the focus rule selects carries no `@lyx_unseen`.
The multi-client warning bullet's "It never blocks, never changes the argv" says another attached client, or a failed client listing, drops the attach selection from the argv.
The Detached windows sentence "one chained after a detached new-window without a target lands on the current window, the strands' window" drops "the strands' window", since attach and the focus rules select agent and overview windows, so the current window can be any window.

`newAgentWindowSelected` and `relevantWindow` are pinned through their callers below, not by pure rows.
`TestAttachArgv_SelectsTheRelevantWindow` drives `AttachArgv` against the fake tmux, scripting `list-windows`, the session pane listing (`answerWindowPanes`) and `list-clients`, with rows: the chained argv ends with `select-window -t` the newest live agent window; with the newest agent's pane dead it targets the older live agent; with no live agent it targets the overview window; with neither no `select-window` is appended; with another client listed none is appended; an unknown client size (`AttachArgv(0, 0)`) gives the bare argv plus the selection; a session whose only pane is Selvage gives the bare argv plus the selection; a `#{window-size}` readback other than `latest` and an unrecognized `#{status}` readback each give the bare argv plus the selection.
`TestAttachArgv_MultiClientWarning`'s rows keep their warnings; its "ListClientsError" row, the one row for a failed client listing, keeps its warning assertion and now seeds the state with a live own-window strand and scripts `list-windows` and the session pane listing (`answerWindowPanes`) the way `TestAttachArgv_SelectsTheRelevantWindow` does, so `relevantWindow` would return that strand's window, and asserts no `select-window` is appended;
the row, `ListClientsError_WarnsAndDoesNotChangeBehaviour` in `internal/reedengine/attach_test.go`, is renamed `ListClientsError_WarnsAndAppendsNoSelection`.
Without the listing failure the same fixture appends the selection, so a missing guard fails the row.
`TestOwnWindow_RealTmux`'s row "RemoveMovesAViewerOnTheEndingWindowToWindowZero" becomes "RemoveMovesAViewerOnTheEndingWindowToTheNewestLiveAgent": with an older live agent window, removing the viewed one selects the older one, never the shell; its other rows are checked against the new rule, which keeps a viewer on the shell or a live agent window.
`TestWindowOrder_RealTmux` gains rows: a new agent window is selected from the overview and from an all-dead agent window, is not selected from a live agent window or the shell and carries `@lyx_unseen` there, and a `Focus: true` own-window strand is selected from the shell; removing the viewed agent window when no live agent is left selects the overview (the newest-live-agent case is `TestOwnWindow_RealTmux`'s row); replacing the viewed agent while an older agent is live leaves the viewer on the older agent's window; reconcile ending a viewed orphan own window selects the newest live agent; one seat leaving a viewed shared window of two switches nothing.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/reedengine` from 265 to 266.

**ImpactSummary:** Three automatic window switches change, all inside `reedengine`: `selectStrandsWindowIfCurrentLocked`'s two callers move to its successor on this card, and `warnMismatchedClientsLocked`'s one caller is `AttachArgv`.
