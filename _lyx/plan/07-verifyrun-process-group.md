# Card 7 — verifyrun kills the verify shell's process group

**Create:**
- `internal/verifyrun/procgroup_unix.go`
- `internal/verifyrun/procgroup_windows.go`

**Edit:**
- `internal/verifyrun/run.go`
- `internal/verifyrun/run_integration_test.go`

**Intent:**
On Unix, `verifyrun` starts the shell in its own process group and, on context cancel or expiry, kills the whole group, not only the shell (D3, #420).
On Windows it kills the shell only, as today, and logs that only the shell was killed.

Two per-OS files hold the mechanics, using only the standard library's `syscall` (`PATTERN-shell-mechanics-seam`: not in `internal/shell`, not through `internal/proc`):

- `procgroup_unix.go` (`//go:build !windows`) sets `SysProcAttr.Setpgid` and installs a `cmd.Cancel` that sends `SIGKILL` to the negative pid (the group), falling back to killing the shell when the group kill fails, and logs the group kill through `internal/logger` with the pid and command (`PATTERN-spawn-observability`).
- `procgroup_windows.go` (`//go:build windows`) installs a `cmd.Cancel` that kills the shell process and logs a Warn on every such kill saying only the shell was killed and its descendants may outlive it, since `verifyrun` cannot tell whether a descendant exists.

`run` calls the per-OS setup on the command before starting it; `cmd.WaitDelay` stays as the bound on the pipe read on both platforms, and the exit-code and error mapping is unchanged (a cancelled ctx still returns `(-1, err)` wrapping `ctx.Err()`).
The package comment's import sentence stays true: standard library and `internal/logger` only.
Bound: the group kill reaches only processes in the verify shell's own group.

`run_integration_test.go` gains the Unix case the acceptance criteria name: a command that backgrounds a long `sleep` holding the output pipe and writes its pid to a file, run under a context that expires quickly with a short wait delay; after `run` returns, the sleep's pid is dead within a bounded poll.
The case skips on Windows with a reason, where the existing shell-only behaviour is unchanged.

**ImpactSummary:** Both verifyrun callers (verifytree's plan verify and webster's per-card verify rerun) now kill the verify command's whole group on cancel or expiry on Unix.
