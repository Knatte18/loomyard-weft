# A stray bash window appeared in the prime reed session at a prime bring-up

`prime-stray-bash-window` · Note · bug, reed, orch

## Seen (operator and orch, 2026-11-05)

The prime's reed session held a window named `bash` (window 1, pane `%1984`) running a plain idle `/usr/bin/bash`, bound to no strand and not reed's `shell` window, which holds Selvage at the end of the window list.
Its shell started at 16:17:38 UTC; the prime overview `beam: prime` (`%1985`) and the orch pane (`%1986`) started at 16:18:15 and 16:18:17, so it came with a prime bring-up or relaunch of those two.
No trace log under `.lym/logs` names the pane; a hand `prefix c` in tmux would give the same window, so the cause is not settled.
The orch closed it on the operator's word.

## Idea

Find which path opens a window with no command at a prime bring-up or a relaunch of the overview or orch, such as a `new-window` before a `respawn-pane` that a race or failure leaves behind, and have reed close or reuse it; a reconcile could also close any unbound plain-shell window other than the shell window.
