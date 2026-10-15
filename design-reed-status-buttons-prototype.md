# Prototype reed status buttons in a throwaway tmux server

`reed-status-buttons-prototype` · Note · audit, reed

## Goal

Try the placement options of note `reed-status-buttons` by hand before that note becomes a task.

## Operator decision (2026-10-15)

A fresh agent builds the prototype, but not now: it starts when the operator says so.

## Scope

- A separate throwaway tmux server (its own `-L` socket), never reed's server and never the prime's session `loomyard`.
- A status bar shaped like reed's two lines, with:
  - one menu button that opens a `display-menu` of sample actions (option 1 in `reed-status-buttons`),
  - a follow toggle button (note `viewer-off-shell`),
  - a third line of action buttons, the destructive ones behind `confirm-before` (option 2).
- The sample actions only echo or display a message; none runs a real `lyx` verb.
- The agent opens the server in a Konsole window for the operator to click through, and tears the server down when the operator is done.
- The tmux config used goes into git under `docs/research/` through a docs-only commit, so the chosen layout can seed the task.
