---
round: 1
focus:
  - "internal/reedengine/lifecycle.go `logBootWindowLocked`: does the display-message target reach the freshly booted session's one window on tmux and psmux, and does a failed read stay non-fatal on every boot path?"
  - "internal/reedengine/window.go `findWindowLocked` and `OpenWindow`: the @lym_hidden filter inverts the old @lym_own skip; settle whether every window OpenWindow opens carries @lym_hidden from creation, so a failure between new-window and set-option cannot leave a window the next call neither reuses nor replaces."
  - "Decision record's first Wanted bullet: does the diff find and fix a path that leaves a window with no command at prime bring-up or at an overview/orch relaunch, or does it show the code has none? Check the commit message and `_lym/landing/summary.md` for the claim the third bullet requires when no such path exists."
  - "Third Wanted bullet: confirm every window reed opens in the prime logs an Info line with window id, pane id and name (boot, own window, break-out, detached window), and that no opening path is left unlogged."
  - "Reed closes only windows it opened: check the reconcile and kill paths against @lym_own and @lym_hidden for any place that still ends or adopts a window by name alone."
  - "internal/reedengine/window_test.go: the reworked OpenWindow cases against the new filter, and whether the boot-window log and the added log fields have coverage."
---

Seed focus only; no prior round exists.
The decision record carries a Goal and no plan, so the Goal is the measuring stick.
