# Each batten the orch starts runs in its own tmux window

`batten-windows` · Note · feature, reed, batten, orch

Seen 2026-10-03: the hub orch starts each batten detached with `setsid nohup lyx batten run <slug>`, so nothing on screen shows what the orch has running; Claude Code's footer shows only "1 shell" for a background helper, which says nothing about what it does.
Idea (operator): when the orch starts a batten, it lands in its own tmux window in the orch's session, so the window list names every run the orch has in flight.
Related notes: `runs-overview` (one status line per run, in its own window) and `orch-owns-the-loop` (lyx starts batten detached, leaves the view undecided); `manifest/designs/reed-multi-window.md` sketches reed's own-window anchoring.
