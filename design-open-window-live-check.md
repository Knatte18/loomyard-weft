# Open-Window opens a window whenever no terminal is attached here, instead of a recorded done

`open-window-live-check` · Note · enhancement, batten, reed

## Seen

Batten's `Open-Window` row is recorded done once per run, so a run that goes on where its window is gone opens none.
Issue #742 (folded into `machine-handover`, point 5) proposes counting the row per machine.

## Idea

Remember nothing: when a run starts or resumes, check live whether any tmux client is attached to the task worktree's reed session on this machine, and open the window when none is.
This covers more than a per-machine record does: a run adopted from another machine, a machine restart that closed every window, and a window the operator closed by mistake.

`machine-handover` builds point 5 per machine as scoped (operator, 2026-10-22: no scope change during Webster); this note replaces that record with the live check afterwards.
Note `open-window-toggle` adds an operator switch that decides whether the window opens at all.
