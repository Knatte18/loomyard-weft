# Card 5 — window-order-and-shell

**Create:**
- `plan:internal/reedengine#sessionWindow` -> `type sessionWindow struct`
- `plan:internal/reedengine#sessionWindowFormat` -> `const sessionWindowFormat = "#{window_id} #{window_index} #{?#{@lyx_hidden},1,0} #{?#{@lyx_shell_check_failed},1,0} #{window_name}"`
- `plan:internal/reedengine#parseSessionWindows` -> `func parseSessionWindows(out string) []sessionWindow`
- `plan:internal/reedengine#TmuxCmd.listSessionWindows` -> `func (p TmuxCmd) listSessionWindows(session string) ([]sessionWindow, error)`
- `plan:internal/reedengine#shellWindowOf` -> `func shellWindowOf(st *ReedState, windowByPane map[string]string) (string, bool)`
- `plan:internal/reedengine#ownWindowInsertArgs` -> `func ownWindowInsertArgs(windows []sessionWindow, shellWindow, previousWindow string) []string`
- `plan:internal/reedengine#hiddenWindowInsertArgs` -> `func hiddenWindowInsertArgs(windows []sessionWindow) []string`
- `plan:internal/reedengine#planShellWindowFix` -> `func planShellWindowFix(windows []sessionWindow, shellWindow string) (rename bool, after string)`
- `plan:internal/reedengine#Engine.ensureShellWindowLocked` -> `func (e *Engine) ensureShellWindowLocked(st *ReedState)`
- `plan:internal/reedengine#Engine.newWindowLocked` -> `func (e *Engine) newWindowLocked(placement []string, argv []string) (string, error)`
- `plan:internal/reedengine#TestWindowOrder` -> `func TestWindowOrder(t *testing.T)`
- `plan:internal/reedengine#TestEnsureShellWindow` -> `func TestEnsureShellWindow(t *testing.T)`
- `plan:internal/reedengine#TestWindowOrder_RealTmux` -> `func TestWindowOrder_RealTmux(t *testing.T)`

**Edit:**
- `internal/reedengine#Engine.openOwnWindowLocked` -> `func (e *Engine) openOwnWindowLocked(st *ReedState, s *Strand, name, group string) (string, error)`
- `internal/reedengine#Engine.createOwnWindowPaneLocked`
- `internal/reedengine#Engine.OpenWindow`
- `internal/reedengine#Engine.AttachArgv`
- `internal/reedengine#Engine.markResolvedStrandWindowLocked`
- `internal/reedengine#TestOpenWindow`
- `internal/reedengine#TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Fix the session's window order as overview, agents in start order, shell last among visible windows, with hidden windows after the shell (D2), name Selvage's window `shell` (D3), and bring a live session booted by an older `lyx` into that order (D10).
The new members live in a new file, `internal/reedengine/windoworder.go`, package `reedengine`; the tests live in `windoworder_test.go` (package `reedengine`) and `windoworder_integration_test.go` (`//go:build tmux`, package `reedengine`).

`listSessionWindows` runs `list-windows -t <exact session target> -F sessionWindowFormat` and `parseSessionWindows` returns one `sessionWindow` per line in index order: id, index, the `@lyx_hidden` flag, the session's shell-check episode flag (a session option, so every line reads the same value) and the name; a line it cannot parse is skipped.
`shellWindowOf` returns the window of `st.SelvagePaneID`, alive or dead, from the session-wide `sessionPaneWindowFormat` listing, and false when no Selvage pane is recorded or the listing does not carry it; it never falls back to `strandWindowTarget`'s other branches, so a session resolved only through a `top` or `below-parent` strand or the current window has no shell window here.

Placement, which `newWindowLocked` applies as `new-window -d` plus the placement arguments plus the caller's own arguments:
`ownWindowInsertArgs` returns `-a -t <previous window id>` when a relaunch's old window is listed, so a relaunch keeps its window's place and the old window ends behind it; otherwise `-b -t <shell window id>` when the shell window is listed, else `-a -t <highest-index non-hidden window>`, else nothing (the plain form);
`hiddenWindowInsertArgs` returns `-a -t <highest-index window>`, hidden windows included, read at open time under the op lock, else nothing.
A refused placed `new-window` on Windows (psmux) logs at Debug and is retried once in the plain form, so the order is best-effort there and nothing fails; on every other platform the error is returned as today.
A missing shell window, which makes the own-window placement fall back, logs at Debug.
The `new-window` literal stays in `spawn.go` and `window.go`, where `panebin_enforcement_test.go`'s allowlist admits it: `newWindowLocked` lives in `spawn.go` beside `openOwnWindowLocked`.

`openOwnWindowLocked` takes `st`, runs `ensureShellWindowLocked(st)` first, lists the windows and opens through `newWindowLocked` with `ownWindowInsertArgs`, so each new agent window lands after every earlier one and before the shell.
For a relaunch, `s.PaneID` still names the old pane when `openOwnWindowLocked` runs (`launchStrandLocked` rebinds it afterwards and `killReplacedOwnWindowPaneLocked` then kills the old pane), so the old pane's window from the session pane listing is the `previousWindow` that keeps the relaunch in place, as D2 requires; a replace removes the old strand first, so its new window opens before the shell as the newest.
`createOwnWindowPaneLocked`'s three calls pass `st` along.
`OpenWindow` reads the state with `LoadState` (a failure logs at Debug and checks nothing), runs `ensureShellWindowLocked` and opens its window with `hiddenWindowInsertArgs`, so every hidden window sits after the shell; its chained `set-option` and `rename-window` commands are unchanged.

`planShellWindowFix` decides the shell check: nothing at all when the shell window is absent from the window listing (a listing that failed or came back empty), logged at Debug by the caller; otherwise rename when the shell window's name is not `shell`, and `after` is the id of the highest-index non-hidden window when any non-hidden window has a higher index than the shell, empty otherwise.
`ensureShellWindowLocked` resolves the shell window, logs at Debug and does nothing when none resolves, and otherwise runs `rename-window -t <id> shell` with `set-option -w -t <id> automatic-rename off`, then `move-window -a -s <id> -t <after>`, each only when the plan asks for it.
A failed window listing logs at Debug and checks nothing, since it cannot read the episode option.
A failed rename or move is logged once per failure episode: at Warn when the listing shows no open episode, after which it sets the session option `@lyx_shell_check_failed`; at Debug while the episode stays open; and on Windows at Debug only.
A check that finds the option set and succeeds logs at Info and unsets it; a check never fails its caller and is retried at the next open or attach.
Bound: the check renames and moves only the window holding the recorded Selvage pane, and only when its name or position is wrong; strands already placed stay where they are, so `Resume` relaunches an in-flight `below-parent` strand in the shell window.
`markResolvedStrandWindowLocked`, which `upLocked` and `Resume` run once Selvage exists, calls the check after the mark, which is how a fresh boot names its one window `shell` (D3).

`AttachArgv` takes the op lock even when no client size is known: inside it, it requires the session, loads the state and runs `ensureShellWindowLocked` ahead of every layout guard, and only then returns the bare argv for an unknown client size, logged at Warn as today.
Taking the lock, reading the session or loading the state are the failures that skip the check, each logged at Warn as today.
Its doc comment drops "returns the bare argv immediately without taking the lock" and says what the pre-flight now does first.
`internal/reedengine/doc.go` describes the window order, the placement flags and their psmux fallback, the `shell` window, and the old-session check with its episode logging.

`TestWindowOrder` is the pure table over `parseSessionWindows`, `ownWindowInsertArgs` and `hiddenWindowInsertArgs`, the placement no engine row asserts by argv, with rows: a listed old window gives `-a -t <old window>` even with a shell listed; a listed shell gives `-b -t <shell>`; no shell gives `-a -t` the highest non-hidden window; no window gives nothing; the hidden placement targets the highest window even when it is hidden.
`TestEnsureShellWindow` drives the engine through its callers, `OpenWindow` and `AttachArgv`, against the fake tmux, scripting `list-windows` and the session listing, with rows: no recorded Selvage pane makes neither `OpenWindow` nor `AttachArgv` issue a `rename-window` or `move-window`; a recorded Selvage pane the session listing does not carry issues neither; a shell window absent from the `list-windows` answer issues neither; a misnamed shell gets the rename and the `automatic-rename off` pin; a shell before a visible window gets `move-window -a -s <shell> -t <last visible>`; a correct shell after every visible window and before hidden ones issues neither; a failed rename logs one Warn line (`logcapture.Capture`) and sets the episode option; a failed rename with the episode option already set logs no Warn; a succeeding check with the option set logs at Info (`logcapture.CaptureVerbose`) and unsets it; `AttachArgv(0, 0)` and `AttachArgv` over a session whose only pane is Selvage each still issue the rename and the move.
`TestOpenWindow` gains the row "AHiddenWindowOpensAfterTheHighestWindow", scripting `list-windows` and wanting `-a -t <highest id>` in the argv, and its argv check finds the temporary name after `-n` instead of at a fixed index.
`TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv`'s comment drops "without taking the lock"; its rows keep wanting the bare argv.
`TestWindowOrder_RealTmux` runs on one cold session.
Like `TestOwnWindow_RealTmux`, it injects through `withInjectedExecutablePath` a `lyx` stub that sleeps for its first argument's seconds, so an `OpenWindow` window stays open, which makes the test process-global and never parallel.
Its rows: after `Up` the session's one window is named `shell`; three own-window strands added in turn hold ascending indexes before the shell; removing the middle one and adding a fourth keeps start order with the shell last; relaunching a dead agent through `AddStrand` with `IfAbsent` keeps its window between its neighbours; two `OpenWindow` windows sit after the shell; a session made old by renaming the shell window and opening a plain window after it is renamed and moved by the next `OpenWindow`; a `below-parent` strand relaunched by `Resume` after a server restart is back in the shell window.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/reedengine` from 262 to 265 for the three new top-level tests.

**ImpactSummary:** Every own window and hidden window reed opens gets an explicit position, and attach, boot and each window open may rename and move the Selvage window; `openOwnWindowLocked`'s one caller, `createOwnWindowPaneLocked`, moves with it, and no package outside `reedengine` changes.
