# Study: close the bus and tmux exposures without a kernel newer than 7.0

Adds `docs/research/platform/sandbox-without-new-kernel-2026-11-23.md`.
The report weighs what closes the two exposures the Landlock report left open on ABI 8,
on kernel 7.0 with no root at run time and no user namespace:
agents reaching the session bus, and through `systemd-run --user` leaving lym's tree,
and agents reaching other hubs' tmux servers.
It adds the two probes it measures with on Neve, `seccomp-connect-probe/main.go` and `same-uid-probe/main.go`,
both `//go:build ignore` so no package build sees them, and a line in `docs/research/README.md`.
No lym code changes.

## Findings

- A seccomp filter that hands every `connect()` to a supervisor closes both exposures, measured on Neve.
  The supervisor reads the address, resolves a path through `O_PATH`, checks its canonical path against an allow list,
  and connects the agent's own socket itself through `pidfd_getfd`.
  Under it the bus, `systemd-run --user`, another hub's tmux and creel sockets, the SSH agent,
  VS Code's IPC socket and abstract X11 were refused.
  The hub's own tmux and creel, git, a Go build, `lym board list` and `claude --version` worked.
- An address swapped by another thread reached the bus 42 times in 200 connects unfiltered, and 0 times filtered.
- A connect costs 26.6 µs and 29.9 µs filtered against 4.0 µs and 2.8 µs unfiltered, over two runs.
- Under Yama `ptrace_scope` 1, an orphan's connect is refused unless the supervisor is a subreaper (measured).
  A closed listener makes every connect fail with `ENOSYS` (measured), so the filter fails closed.
- `SO_PEERCRED` names the supervisor for the connects it makes, while `SCM_CREDENTIALS` still names the agent (measured).
  creel's sender identity, which reads `SO_PEERCRED`, would have to move.
- Landlock ABI 8 stacks under the filter, and a tmux server started inside both allocated panes and was refused the bus.
- Taking the sockets away closes neither exposure, measured.
  A hard-coded bus address undoes an emptied environment.
  A socket or directory's owner sets a mode of 000 back, inside a Landlock domain too,
  and `/proc/net/unix` lists every socket path.
- A separate Unix user and an AppArmor profile per hub are judged from documentation, both needing root at setup.

The report ranks: shellguard PreToolUse rows now;
the Landlock-plus-seccomp area when a barrier is wanted, after the probe runs on OSL-1046;
AppArmor per hub as the fallback; take-away and a separate user left out.
It lists the tasks the area would take.

## Merge danger

**Door:** two-way
**Blast radius:** docs
