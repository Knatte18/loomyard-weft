# Card 12 — run-window-close

**Create:**
- `internal/termwindow#pairGoneWaitSeconds`
- `internal/termwindow#attachLineWaiting`
- `internal/termwindow/termwindow_integration_test.go`
- `internal/reedengine#TestAttachGeometry_ClientExitStatusOnSessionEndAndServerShutdown`

**Edit:**
- `internal/termwindow#attachLine` -> `func attachLine(lyxPath, dir string) string`
- `internal/termwindow#Launcher.OpenAttach`
- `internal/termwindow#TestLauncher_ArgumentsOpenTitledWindowRunningAttach`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Build D10: the `lyx batten run --window` terminal window closes when its run ends.

`attachLine(lyxPath, dir)` is `attachLineWaiting(lyxPath, dir, pairGoneWaitSeconds)`, and `attachLineWaiting` builds one POSIX line through `shell.Posix()` (`Invoke`, `Quote`, `Chain`), as the line is built today (`PATTERN-shell-mechanics-seam`): run `<lyx> reed attach` and keep its status; on 0, `exit 0`; otherwise print the status and that the window waits up to `waitSeconds` seconds for the run's worktree to be removed, poll once a second for that long and `exit 0` as soon as the quoted `dir` is no longer a directory, and when the wait ends with the directory present, `exec bash`, keeping an interactive shell and the attach's output.
Bound: the window ends only on a 0 attach or on the pair directory gone within the wait.
The same line serves Konsole and Windows Terminal, whose argument builder still escapes each `;`.
An operator detach, and a pair session ended while its run lives (`lyx reed down`, a watchdog reap), end the attach with 0 and close the window too, as D10 accepts; the run stays reachable from the prime.
`OpenAttach` passes its `dir` and its doc comment, with `attachLine`'s and the package comment, says when the window closes.
`docs/overview.md`'s batten `--window` sentence says the window closes when the run's session ends or its worktree is removed.

The open risk on tmux's attach exit status is verified on the installed tmux by `TestAttachGeometry_ClientExitStatusOnSessionEndAndServerShutdown` (new, tmux tier, `attachgeometry_integration_test.go`): over the attach-geometry fixture plus a second session created up front (so killing the first session leaves the server running under tmux's `exit-empty`), a client attached to the first session through `startInPTY` exits 0 when that session is killed, and a client attached to the second exits non-zero when the server is killed, each read from its pty child's `Wait`.

Tests:
- `TestLauncher_ArgumentsOpenTitledWindowRunningAttach` pins the exact line for a lyx path and a directory, both needing quoting, and both argument builders around it, Windows Terminal's with each `;` escaped.
- `termwindow_integration_test.go` (`//go:build integration`, it spawns bash): `TestAttachLine_EndsOrKeepsTheShellByAttachStatus`, table-driven, runs `bash -c` over `attachLineWaiting` with a stub executable script standing in for lyx and a one-second wait, stdin feeding `echo kept-shell`: a stub exiting 0 ends without `kept-shell`; a stub exiting 3 with the directory absent prints the status and ends without `kept-shell`; a stub exiting 3 with the directory present prints the status and then `kept-shell` from the kept shell; and, with a three-second wait, a stub that starts a background `sleep 1` then `rmdir` of the directory and exits 3 prints the status and ends without `kept-shell`, the directory removed during the wait.
- `test-budget.yaml`: `internal/termwindow` 2 → 3, `internal/reedengine` 256 → 257.

**ImpactSummary:** termwindow's one window line and its one caller; one tmux-tier reed test; no other package builds the line.
