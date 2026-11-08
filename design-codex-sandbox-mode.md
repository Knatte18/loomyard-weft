# Codex engine: the forced workspace-write sandbox fails where bubblewrap cannot create user namespaces

`codex-sandbox-mode` · Task · bug, codex, shuttle, preflight · running · Describe

Imported from [issue #83](https://github.com/Knatte18/loomyard/issues/83).

Every Codex launch from `lym shuttle` passes `-a never -s workspace-write` (hardcoded in `internal/shuttleengine/codexengine/command.go`, the launch line). On Ubuntu with `kernel.apparmor_restrict_unprivileged_userns = 1` and no sudo to add an AppArmor profile for bwrap, that sandbox cannot start: every shell command fails with `bwrap: setting up uid map: Permission denied`, the session stalls (shuttle state `idle-stalled`, cause `no-output`) or dies (`codex-exit`), and no output file is ever written.

The mode cannot be changed by configuration: `-s` on the command line overrides a `sandbox_mode` in a project `.codex/config.toml`, the codex module has no sandbox key, and `binary` may not be set in a machine or local layer.

Current workaround: a wrapper set as `binary` in the worktree layer of `_lym/config/codex.yaml`, which rewrites `-s workspace-write` to `-s danger-full-access` and execs codex.

Wanted, either or both:
- a `sandbox_mode` key in the codex module (workspace-write by default, danger-full-access allowed), settable per machine;
- a preflight that runs bwrap once and, when it cannot create a user namespace, fails the run at once with that cause and the way forward, instead of a session that stalls without output.

Environment: Linux 7.0.0-42-generic (Ubuntu), codex-cli 0.163.0.