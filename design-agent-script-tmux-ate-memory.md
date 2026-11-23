# A darn writer ran script -qfc tmux attach outside any memory cap and froze Neve

`agent-script-tmux-ate-memory` · Note · bug, process, shuttle

## Seen

Neve froze at 20:20 on 2026-11-23 and was reset by the power key at 20:20:23.

The memsample journal line at 20:20:05 shows `script(2)=6185M`, with the user slice at its 16 GB cap and swap at 4 GB:

- pid 424182, age 3 s, rss 3583M: `script -qfc tmux -S /home/knatte/Code/loomyard-LYMHUB/rc-click-wrong-client-session/.scratch/tm/sock attach -t beta /dev/null`
- pid 424180, age 3 s, rss 2679M: the same command for session `alpha`

The Darn-Write agent of `rc-click-wrong-client-session` built its own tmux server under its worktree's `.scratch/tm` to reproduce the multi-client RC bug, and ran `(script -qfc "tmux -S $S attach -t alpha" /dev/null </dev/zero >/dev/null 2>&1 &)`, the same for `beta`.
With stdin on `/dev/zero`, `script` reads zero bytes without end and queues them in memory for a tmux client that cannot take them, about 1 GB a second each.
The subshell detached both, so they outlived the agent's Bash call.
Both had systemd's user manager as parent, so they ran outside the 2 GiB memory scope lym puts each test binary in.
The earlier OOM kills at 20:13 and 20:14 (`pkg.test`, `proc.test`, `testcover.test`) were contained by those scopes and did not freeze the machine.

## Idea

An agent's ad hoc shell command can start any process outside a memory cap.
Two fixes to weigh:

1. `internal/shellguard` refuses `script` and any raw `tmux` command in a run agent's Bash, as the orch's own session already refuses `tmux`; a repro that needs tmux goes through a tmux-tier test, which runs capped.
2. Every agent session and its children run inside a memory-capped scope, so a runaway command is OOM-killed in its scope instead of freezing the machine; this overlaps the sandbox study `sandbox-without-new-kernel`.
