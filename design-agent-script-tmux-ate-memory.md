# A darn writer ran script -qfc tmux attach outside any memory cap and froze Neve

`agent-script-tmux-ate-memory` · Task · bug, process, shuttle

## Seen

Neve froze at 20:20 on 2026-11-23 and was reset by the power key at 20:20:23.

The memsample journal line at 20:20:05 shows `script(2)=6185M`, with the user slice at its 16 GB cap and swap at 4 GB:

- pid 424182, age 3 s, rss 3583M: `script -qfc tmux -S /home/knatte/Code/loomyard-LYMHUB/rc-click-wrong-client-session/.scratch/tm/sock attach -t beta /dev/null`
- pid 424180, age 3 s, rss 2679M: the same command for session `alpha`

The Darn-Write agent of `rc-click-wrong-client-session` wanted to learn how tmux resolves `display-message -c <client> -t ...` with two attached clients before changing `clientAgentPaneVia`.
It built its own tmux server under its worktree's `.scratch/tm` and ran `(script -qfc "tmux -S $S attach -t alpha" /dev/null </dev/zero >/dev/null 2>&1 &)`, the same for `beta`.
With stdin on `/dev/zero`, `script` reads zero bytes without end and queues them in memory for a tmux client that cannot take them, about 1 GB a second each.
The subshell detached both to systemd's user manager, so they outlived the agent's Bash call and ran outside the 2 GiB memory scope lym puts each test binary in.
The earlier OOM kills at 20:13 and 20:14 (`pkg.test`, `proc.test`, `testcover.test`) were contained by those scopes and did not freeze the machine.

## Why the agent did it

- The safe path is closed and the dangerous one open: a run agent is refused tmux-tier `lym gate test` runs, which run capped against a proper tmux, but raw `tmux` and `script` in its Bash are not refused.
- No instruction tells a writer how to learn how an external tool such as tmux behaves, so it improvised in the shell.

## Fix (operator, 2026-11-23)

1. `internal/shellguard` refuses `script` and any raw `tmux` command in every run agent's Bash (writers, fixers, reviewers, drivers, every engine), as the orch's own session already refuses `tmux`.
   The refusal names the way forward: write a tmux-tier test in the repository and run it through `lym gate test --tier`.
2. A run agent whose change needs a fact about tmux behaviour can run a tmux-tier test it writes, through `lym gate test --tier tmux --card` or the form that fits, instead of being refused tmux-tier runs outright.
   Discussion settles how narrow that opening is, so a run cannot start the whole tmux tier.
3. The writer and fixer stencils say: learn how an external tool behaves from a test in the repository under the gate, never from ad hoc processes in the shell; never start a background or detached process from Bash.
