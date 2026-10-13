# reed resize re-apply fails: layout counts one pane fewer than the window

`reed-resize-pane-count` · Note · bug, reed

## Seen (2026-10-13)

The hub reed watchdog's trace file (`_board/.lyx/logs/trace-20261008T181526Z-f2e3f7df172f3d30-3854.log`) holds 25,343 lines of `reed: resize re-apply failed` (`internal/reedengine/watchloop.go:437`) since 2026-10-08, still firing on 10-13 09:36:

`session=go-driver-assistant err="select-layout: exit status 1: have 4 panes but need 3: …"`

The layout reed re-applies on resize counts one pane fewer than the window holds, so tmux refuses it on every tick.
The window's layout is then never restored after a resize.
The failure predates `reed-multi-window` (landed 2026-10-13), which moved strands into their own windows; whether that landing changes or removes the mismatch is unchecked.

## To find

- Which pane the layout leaves out: the selvage pane, a status pane, or a strand pane that the recorded state lost.
- Whether reed's recorded pane list and the live window disagree, and how they came apart.

The per-tick WARN with no dedupe is `log-level-hygiene`'s part; this note is the layout bug itself.
