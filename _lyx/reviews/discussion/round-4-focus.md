---
round: 4
focus:
  - "D10 (decision-record.md) logs a failed window rename or move: settle whether its Warn, Debug and recovery levels and the session option that records an open failure episode match the `internal/logger` Level policy (Warn once per episode, Debug while it persists, Info at recovery), and whether the Constraints line citing `PATTERN-log-levels` is true of it."
  - "D3 and D10: `strandWindowTarget` in `internal/reedengine/windowtarget.go` resolves the Selvage pane's window, then a non-own-window strand's window in window 0, then the session's current window. Settle whether D10's rename-and-move, run before every window open and in every attach, can act on the overview window or an agent window when the Selvage pane is absent, and whether the stated bound holds."
  - "D5's Attach bullet appends a `select-window` to the attach argv: settle whether it can move a second attached client off the shell or a live agent window, given that tmux keeps one current window per session, and whether that agrees with D5's Bound and the shell-counts-as-watched rule."
  - "Acceptance criteria: settle whether the migration criterion forces the rename and move to run when Selvage is the only pane in the strands' window and when no client size is known, as D10's placement ahead of the layout guards intends."
---

Focus targets the sites round 3 reopened; the closed round 1 and round 2 questions are not repeated.
