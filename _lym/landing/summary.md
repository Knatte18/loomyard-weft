# Cap every strand pane's memory and tell the parent orch of an OOM kill

A process an agent starts in its pane used to run with no memory bound.
tmux puts each pane in a `tmux-spawn-<uuid>.scope` of its own, and that scope sets no `MemoryMax`.
So the two detached `script -qfc "tmux attach"` processes that read `/dev/zero` on 2026-11-23 grew to 6 GB in 3 seconds and froze Neve.
Every strand pane now runs in a capped scope, so the kernel kills such a process inside the pane's scope, and the pane and the machine keep running.

## What changes

- **The cap.** `launchStrandLocked` is the chokepoint every strand launch goes through.
  It now reads the fresh pane's shell pid and moves that shell into a transient user scope, `lym-pane-<pid>.scope`, before it types the launch.
  The scope sets `MemoryMax` to the cap, `MemorySwapMax=0`, `OOMPolicy=continue` and `CollectMode=inactive-or-failed`.
  Every process the launch starts inherits the scope, a detached or reparented child included.
  A kill at the cap ends only the process the kernel picks, so the agent's session and its pane live on.
- **The config value.** The cap comes from a new `reed.yaml` key, `pane_memory_max_mib`, with a default of 8192.
  A value of 0 or less takes the default, as `operator_quiet_s` does.
- **The warning.** A pane the cap cannot reach still launches, with a `Warn` in the hub log naming the cause.
  Causes include no `busctl`, a user manager that refuses the scope, one that holds no memory controller for its scopes, and Windows.
- **The notice.** On each discovery cycle, the per-hub reed watchdog reads the `oom_kill` count of every strand pane's cgroup.
  A new kill posts a creel notice, signed `REED` and sent through `orchcli.NotifyAsker`, to the worktree's parent orch.
  The notice names the strand, the pane, the worktree and the killed process with its pid, read from the kernel journal.
  When the journal cannot be read, the notice counts the kills and gives the error.
- **`proc` primitives.**
  `CapProcessMemory` starts the scope over D-Bus with `busctl ... StartTransientUnit`, waits until the pid has joined it, and checks its `memory.max`.
  `ProcessOOMKills` reads a pid's cgroup and its `oom_kill` count.
  `OOMKillsInCgroup` reads the `oom-kill:` kernel lines for one cgroup through `journalctl --dmesg --boot --grep`.

## Design decisions

- **Move the pane's shell, not wrap the launch.**
  The pane keeps tmux's default login shell, and the window commands are unchanged.
  The cap does not depend on tmux's own scope, so it holds with a tmux built without systemd support too.
  `systemd-run` cannot adopt a running pid, so the scope is started through `StartTransientUnit` with `PIDs`.
- **Default 8192 MiB.**
  The memsample journal shows agent sessions (`claude`) at a few hundred MB, with peaks of about 1.1 GB over the last week.
  The pane must also hold the builds the agent runs in it, such as `go build` and `go vet` over this module.
  Tests that `lym gate test` runs keep their own 2 GiB `systemd-run` scope, which is a sibling under `app.slice`, not a child of the pane's scope.
  8 GiB leaves room for that work and still stops a runaway at about a quarter of Neve's 30 GB.
- **`OOMPolicy=continue`.** The default policy, `stop`, would end the whole scope on a kill, and with it the agent.
- **Notice from the watchdog.** The watchdog is the one long-lived per-hub process that already walks every session.
  It holds the last count per cgroup in memory and counts a cgroup it sees for the first time from 0.
  So a kill that happens before its first look is still told.
  A daemon that starts afresh tells a pane's earlier kills again.
  That trade puts a missed notice below a repeated one.
- **Only strand panes.** Selvage and the operator's shell window are lym's own and the operator's panes, not agent panes, and stay uncapped.

## Tests

- `proc` (tier 1): a capped `sh` runs a child that passes the cap.
  The child is killed, the shell prints that it lives on, the scope holds the cap, and `ProcessOOMKills` counts one kill.
  `OOMKillsInCgroup` reads the journal without error, and any kill it names there is the child's.
  A second test checks that capping a process that has already ended is an error that wraps `ErrMemoryCap`.
  Tier 0 covers the kernel-line parser and the `memory.events` count.
- `reedengine` (tier 2, the red-first test): `AddStrand` on a real tmux leaves the pane's shell in a cgroup whose `memory.max` is the template's 8192 MiB.
  It uses only the merge base's API, and on the merge base it reads `max`.
- `reedcli` (tier 0): the planner tells a new kill once, counts a first-seen cgroup from 0, skips unreadable and dead panes, and forgets gone cgroups.
  Another test pins the notice line for a named kill, an unnamed kill and an unreadable journal.
- `cmd/lym` spawn-observability allowlist: `internal/proc/processscope_linux.go` is listed beside `memorycap_linux.go`, for the same import-cycle reason.

## Merge danger

**Door:** two-way
**Blast radius:** reed
