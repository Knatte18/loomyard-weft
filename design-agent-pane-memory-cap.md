# Every agent pane runs in a memory-capped scope, so a runaway child is killed there

`agent-pane-memory-cap` · Task · bug, reed, shuttle, process · running · Code-Burler-Fix

## Seen

On 2026-11-23 at 20:20 a Darn-Write agent started two detached `script -qfc "tmux attach"` processes reading `/dev/zero`; they grew to 6 GB in 3 seconds and froze Neve (task `agent-script-tmux-ate-memory` holds the details and the shell-guard fix).
The journal shows each agent pane already runs in a systemd user scope of its own (`tmux-spawn-<uuid>.scope`, "tmux child pane ... launched by process ...").
A detached child stays in its pane's cgroup even when reparented to the user manager, so a memory limit on that scope would have OOM-killed the two processes inside it and left the machine running.
Today only test binaries run under a cap: lym's 2 GiB `systemd-run` scope per test, which contained the OOM kills at 20:13 and 20:14.

## Fix (operator, 2026-11-23: we must have a mechanism that stops such runaway processes)

- Every agent pane lym spawns, in every hub and engine, runs in a systemd user scope with `MemoryMax` and `MemorySwapMax` set, so the kernel kills a runaway process inside that pane's scope before the machine runs out of memory.
- The cap is a hub config value with a default that fits an agent session plus the builds and tests it runs; Discussion picks the default from the memsample journal (`journalctl -t memsample`), where agent sessions run at a few hundred MB.
- A pane whose scope cannot get the cap, such as on a machine without the memory controller delegated to the user manager, starts with a warning naming the cause, not a refusal.
- An OOM kill inside an agent's scope reaches the run's parent as a notice naming the pane and the killed process, so the orch sees it instead of a silent dead strand.
- Works on Ubuntu 26.04 with kernel 7.0 (Neve) and Ubuntu 24.04 (OSL-1046) without root.
