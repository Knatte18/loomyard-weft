# Card 11 — reed-attach-exec

**Edit:**
- `internal/reedcli#reedCLI.attachCmd`
- `internal/reedcli/doc.go`
- `internal/reedcli#TestSmokeLifecycle`
- `cmd/lyx#shortcutProblems`
- `cmd/lyx#TestShortcuts_FileAndRootListing`

**Create:**
- `internal/reedcli#handOver`
- `internal/reedcli/attach_unix.go`
- `internal/reedcli/attach_windows.go`

**Intent:**
On Unix, `lyx reed attach` replaces itself with the tmux client, so no lyx process outlives the handover, and an attach never keeps an old binary running (D9).

`attachCmd`'s pre-flight is unchanged: it ensures the session, checks status, spawns the watchdog and reads the terminal size.
Its terminal-handover tail calls `handOver(c.eng.TmuxPath(), c.eng.AttachArgv(cols, rows))` and sets the exit code it returns, as today's tail sets the child's.
`handOver` is declared once per platform file, both `package reedcli`:
- `attach_unix.go` (`//go:build !windows`) logs the exec at Info, naming tmux and the size, before it happens, then calls `syscall.Exec` with the tmux path, an argv of the tmux path followed by `args`, and `os.Environ()`.
  The exec skips the root's post-run, which is why the log line comes first (`PATTERN-spawn-observability`); the logger writes each line as it is logged, so nothing waits in a buffer.
  The process's run directory is left to the next start's sweep, as for the status panes' re-exec.
  A failed exec returns exit code 1 and the error, which the tail logs.
  Under `testing.Testing()` it runs the child path instead, as `selfexec`'s exec seam does, so an in-process test never replaces the test binary.
- `attach_windows.go` (`//go:build windows`) keeps today's child process, with stdio inherited, and its spawn and exit logs.
The interactive-handoff exception of `PATTERN-cli-cobra` already covers `attach`.

Bound: nothing runs after the attach in that process, so `reed attach` must be the last argv of any shortcut.
`shortcutProblems` gains that check: an argv that resolves to `reed attach` and is not the shortcut's last argv is a problem, "shortcut %q runs %v after reed attach, which never returns".
`TestShortcuts_FileAndRootListing` gains the synthetic row "an argv after reed attach", `{{"reed", "attach"}, {"orch", "start"}}`, wanting that problem.
`prime`, which already has `reed attach` last, keeps passing.

`reedcli/doc.go` states that `attach` execs tmux on Unix and keeps a child on Windows.
`attachCmd`'s `Long` says attach replaces itself with `tmux attach-session` on Unix and runs it as a child on Windows, and keeps the rest of its text.

`TestSmokeLifecycle`'s `AttachRendersInsideHarnessPane` subtest, `tmux` tier, reads the harness pane's `#{pane_current_command}` once the marker shows and asserts it reads `tmux`.
With a lyx parent left waiting, it reads the lyx binary's name.
The detach still prints `ATTACH-EXIT:0`, now tmux's own exit code.

**ImpactSummary:** One verb's handover tail moves behind a two-file platform split, Unix exec and Windows child, and the shortcut guard gains one rule the current shortcut file already meets.

**Commit:** 11: reed attach: exec tmux in place on Unix
