# A stray bash window appeared in the prime reed session at a prime bring-up

`prime-stray-bash-window` · Task · bug, reed, orch · running · Finalize

## Seen (operator and orch, 2026-11-05)

The prime's reed session held a window named `bash` (window 1, pane `%1984`) running a plain idle `/usr/bin/bash`, bound to no strand and not reed's `shell` window, which holds Selvage at the end of the window list.
Its shell started at 16:17:38 UTC; the prime overview `beam: prime` (`%1985`) and the orch pane (`%1986`) started at 16:18:15 and 16:18:17, so it came with a prime bring-up or relaunch of those two.
No trace log under `.lym/logs` names the pane; a hand `prefix c` in tmux would give the same window, so the cause is not settled.
The orch closed it on the operator's word.

## Wanted

- Find the path that can leave a window with no command at a prime bring-up or at a relaunch of the overview or the orch, such as a `new-window` before a `respawn-pane` that a race or failure leaves behind, and fix it there.
- Reed closes or reuses only windows it opened itself; a window the operator opened by hand stays.
- If no such path exists in the code, the run says so in its landing and adds a trace line naming every window reed opens in the prime, so a recurrence shows its source.
