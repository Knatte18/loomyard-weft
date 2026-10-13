# reed: strand windows first, Selvage last

`reed-window-order` · Note · bug, reed

## Seen (operator, 2026-10-13)

Since `reed-multi-window` each strand gets its own tmux window, but the session's window 0 holds Selvage alone, and `lyx reed attach` lands there.
A strand such as `discussion` opens in window 1, detached and never selected (`internal/reedengine/doc.go`, "Detached windows" and the own-window strand lines), so the operator attaching to a run sees only Selvage and has to switch windows to see the agent at work.

## Wanted (operator, 2026-10-13)

Selvage goes last.
When a strand's window opens beside Selvage, the strand's window comes first and is the one attach shows by default: the foreground work is what the operator watches.

Alt+Up and Alt+Down do not reach them either: they step between the panes of the strands' window only (`internal/reedengine/bindings.go:71-72`, `select-pane -t :.-`/`:.+`, gated on `@lyx_strands`), and an own-window strand lives outside that window, so from Selvage the keys find nothing to step to (operator, 2026-10-13).

The status bar's VIEW button selects the strands' window and unzooms it (`internal/reedengine/bindings.go:33-36`, `bar.go:27-28`); with every strand in its own window that window holds Selvage alone, so VIEW now shows only the console and its name misleads (operator, 2026-10-13).

The operator wants Selvage's window kept, one per session: it keeps the session alive when every loom pane has gone, and it is always a terminal into the session's worktree (2026-10-13).

Window order today is tmux's creation order: every window takes the next free index (`new-window -t '=<session>:'`, `internal/reedengine/spawn.go:293`), so Selvage's window, made first, is 0, and the window buttons follow index order; no reed code sorts.
Focus today: a new strand window opens with `new-window -d` and never takes focus; its button gets the unseen mark (`@lyx_unseen`, underlined) until selected.
The one automatic switch is back: when the window being viewed ends, `selectStrandsWindowIfCurrentLocked` (`ownwindow.go`) selects the strands' window, which is Selvage.

## Scope to discuss

- A way to put Selvage last: give its window a fixed high index (for instance `move-window -t :99` at session creation), draw its button last, and have attach select the lowest strand window, else Selvage.
- Automatic switching: when the viewed window ends, switch to the next strand window rather than Selvage; whether a new strand window takes focus while the operator views Selvage (never while another strand is being watched).

- VIEW is in practice the Selvage button now. The operator leans to renaming it `shell`, in lower case like the window buttons rather than capitals, and placing it last (2026-10-13).
- Selvage lives in the strands' window, which holds only Selvage once every strand runs in its own window; whether a strand can still land in that window (a window-0 strand) decides whether Selvage needs a window of its own.

- Alt+Up and Alt+Down step across every strand, own-window strands included, with Selvage last.

- Window order: strand windows before Selvage's window, and which strand comes first when several are open (the newest, or the run's current row).
- Whether a newly opened strand window takes focus for an attached client, or only becomes attach's default.
- What attach shows when no strand is open (Selvage).
