# reed: overview window first, agents next, shell last

`reed-window-order` · Note · bug, reed, loom, shed

## Seen (operator, 2026-10-13)

Since `reed-multi-window` each strand gets its own tmux window, but the session's window 0 holds Selvage (and an LLM driver strand, with the loom status pane above it), and `lyx reed attach` lands there.
A strand such as `discussion` opens in window 1, detached and never selected (`internal/reedengine/doc.go`, "Detached windows" and the own-window strand lines), so the operator attaching to a run does not see the agent at work and has to switch windows.

Alt+Up and Alt+Down do not reach the other windows: they step between the panes of the strands' window only (`internal/reedengine/bindings.go:71-72`, `select-pane -t :.-`/`:.+`, gated on `@lyx_strands`).

The status bar's VIEW button selects the strands' window and unzooms it (`internal/reedengine/bindings.go:33-36`, `bar.go:27-28`); with the strands in their own windows that window holds little but Selvage, so its name misleads.

The loom status pane (`lyx loom checklist`) shares window 0 at about five lines, so it shows three rows and "… 23 more rows not shown".
Its wait line says only "wait bouncer Plan-Bouncer" while the run waits on a circling escalation to the parent; `lyx loom status` holds the full reason.
A pane started before `run-window-bugs` landed (`e17b2921`, the status pane clears screen and scrollback on redraw) keeps the old binary and stacks earlier renders in its scrollback until it restarts.

Window order today is tmux's creation order: every window takes the next free index (`new-window -t '=<session>:'`, `internal/reedengine/spawn.go:293`), and the window buttons follow index order; no reed code sorts.
Focus today: a new strand window opens with `new-window -d` and never takes focus; its button gets the unseen mark (`@lyx_unseen`, underlined) until selected.
The one automatic switch: when the window being viewed ends, `selectStrandsWindowIfCurrentLocked` (`ownwindow.go`) selects the strands' window.

## Wanted layout (operator, 2026-10-13)

Windows give more room than panes in one window, so:

- Window 0 is always the overview: a loom or shed status in a window of its own, for any kind of shed run (an LLM-driven or Go-driven loom, darn, batten), showing exactly how far the run has come, with the detail of each row reachable by scrolling on.
- The agents a run spawns, the LLM driver included, each get a window after the overview.
- The last window is always the shell: Selvage, kept one per session, since it keeps the session alive when every loom pane has gone and is always a terminal into the worktree; its button is named `shell`, in lower case like the other window buttons, and replaces VIEW.
- What matters most to the operator is seeing the most relevant running agent at any time.

## Scope to discuss

- Which window attach and automatic switching show: the overview, or the most relevant running agent (the run's current row's agent, else the newest); a new agent window takes focus only when the operator is not watching another agent.
- When the viewed window ends, switch to the next relevant agent window, else the overview, never the shell.
- Keeping the order fixed: the overview at index 0, the shell at a fixed high index (for instance `move-window -t :99`), agents between in start order.
- Alt+Up and Alt+Down step across every window in that order.
- The overview's content: the full row list with time and rounds, the wait reason in full (an escalation waiting on the parent named as such), and how the operator scrolls to a row's detail.
- The status pane restarting on the new binary after a deploy, so a fix reaches runs already in flight.
