# reed: strand windows first, Selvage last

`reed-window-order` · Note · bug, reed

## Seen (operator, 2026-10-13)

Since `reed-multi-window` each strand gets its own tmux window, but the session's window 0 holds Selvage alone, and `lyx reed attach` lands there.
A strand such as `discussion` opens in window 1, detached and never selected (`internal/reedengine/doc.go`, "Detached windows" and the own-window strand lines), so the operator attaching to a run sees only Selvage and has to switch windows to see the agent at work.

## Wanted (operator, 2026-10-13)

Selvage goes last.
When a strand's window opens beside Selvage, the strand's window comes first and is the one attach shows by default: the foreground work is what the operator watches.

Alt+Up and Alt+Down do not reach them either: they step between the panes of the strands' window only (`internal/reedengine/bindings.go:71-72`, `select-pane -t :.-`/`:.+`, gated on `@lyx_strands`), and an own-window strand lives outside that window, so from Selvage the keys find nothing to step to (operator, 2026-10-13).

## Scope to discuss

- Alt+Up and Alt+Down step across every strand, own-window strands included, with Selvage last.

- Window order: strand windows before Selvage's window, and which strand comes first when several are open (the newest, or the run's current row).
- Whether a newly opened strand window takes focus for an attached client, or only becomes attach's default.
- What attach shows when no strand is open (Selvage).
