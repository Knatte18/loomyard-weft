# Card 13 — reed-window-placement

**Create:**
- `internal/reedcli/handterminalwindow_integration_test.go`
- `internal/reedcli#handTerminalWindow`
- `internal/reedcli#TestHandTerminalWindow_DirectoryInNoWorktreeHandsNothing`

**Edit:**
- `internal/reedcli/window.go#`
- `internal/reedcli#reedCLI`
- `internal/reedcli#TestWindowCmd_OpensATerminalAttachedToTheToldWorktree`
- `internal/desktophook/window.go#`
- `internal/desktophook/template.yaml`
- `//docs/overview.md`

**Uses:**
- `internal/kwinplace#OpenWindow`
- `internal/kwinplace#Spawn`
- `internal/reedcli/doc.go`

**Intent:** Make `lym reed window` hand each window to the placement child when D1's pre-launch checks pass (D1, D12).

`internal/reedcli/window.go` changes whole: its file comment, `openTerminalWindow` and `windowCmd`.
The file comment says that under placement the verb hands the window to the placement child, which runs the machine's window command,
and that otherwise the verb runs it through `internal/desktophook` as today.
`openTerminalWindow` becomes `func openTerminalWindow(dir, title, lymPath string) (int, bool, error)`:
it resolves the launcher and opens the window through `kwinplace.OpenWindow`, returning its pid and whether the checks passed.
`reedCLI.openTerminal`, the seam, takes the same signature;
a new seam field `handPlacement func(lymPath, dir, title string, pid int) error`, nil meaning `handTerminalWindow`, joins it.
`handTerminalWindow` resolves the worktree holding `dir` and calls `kwinplace.Spawn` with its hub, name, the title, the pid and whether it is the prime;
a directory in no worktree has no hub, so nothing is handed, it logs at Debug and returns nil.
`windowCmd`:
- with the checks passing, hands the window to `handPlacement` and runs no `placeWindow`, since the child runs the window command;
- a hand-off that fails warns and runs `placeWindow` after the wait, as with the checks failing, since no child exists to run the command; the verb still succeeds (D1);
- with the checks failing, runs `placeWindow` after the wait as today (D12).
Its `Long` help says that under placement the placement child runs `desktop.window_command`, with no wait, except when the child is ended by SIGTERM (Review criterion).
The same sentence goes into `template.yaml`'s `window_command` and `window_wait_ms` comments, into `internal/desktophook/window.go`'s package comment,
and into `docs/overview.md`'s paragraph on `lym reed window` and `desktop.window_wait_ms`.
`docs/overview.md`'s package tree gains a line for `internal/kwinplace/`, the KWin and kscreen boundary and the placement flow, below the `internal/desktophook/` line,
and that line widens to name the placement keys and the cancellable window command beside the window command and the status-bar buttons.

`TestWindowCmd_OpensATerminalAttachedToTheToldWorktree` (`tier0`) stubs the two seams; its existing rows return `false` and keep their assertions, and new rows:
- `PlacementHandsTheWindowToTheChild`: the opener answers pid 4242 and `true`; `handPlacement` is called once with the directory, title and 4242, and `placeWindow` is not called;
- `HandOffFailureRunsTheWindowCommand`: the same with `handPlacement` failing; `placeWindow` is called once with the directory, and the verb's envelope is ok.
They add under 0.1 seconds to one `tier0` run.
`TestHandTerminalWindow_DirectoryInNoWorktreeHandsNothing`, alone in `internal/reedcli/handterminalwindow_integration_test.go`, is `tier1`, since resolving the directory runs git:
a directory under `t.TempDir()` with the lym path `/nonexistent/lym` returns nil, so no `Spawn` ran, which would fail to start that path.
`TestPlaceTerminalWindow` stays as it is, since `placeTerminalWindow` is unchanged.

**ImpactSummary:** `openTerminalWindow` has one caller, `windowCmd`'s default; the batten Open-Window row and its reopen reach the new path through `lym reed window` unchanged.
