# Discussion round 4 — fixer report

All eight findings hold up against the code and against tmux 3.6, and all are fixed.
On a throwaway server, `resize-pane -y` and `select-layout` each cleared `window_zoomed_flag` on a zoomed window, and `run-shell` expanded `#S` and `##` inside a single-quoted token (F1, F3).
`lyx loom validate-discussion` exits 0 after the changes.

## Fixed

- **F1 (BLOCKING, design).**
  D1 now states that a zoom survives reed's own geometry steps.
  Every step that issues `select-layout` or `resize-pane -y` on the strand window (the `window-resized` hook and every layout apply) reads the zoom flag and the zoomed pane first, runs on the unzoomed window, then re-zooms that pane.
  The Decision now has a Bound: a step re-zooms only a window that was zoomed and only that pane, and a pane that is gone leaves the window in the overview.
  A new acceptance criterion covers resize, re-attach, add and resume while zoomed, plus the layout at the next unzoom.
  The support log records the rejected alternative (leave a zoomed window alone) and Q19.
- **F2 (MEDIUM, design).**
  D7 now has reed run the pins itself right after every layout apply, as well as from the hook.
  That run issues only the pin entries, never the watchdog entry, so an apply cannot trigger another apply.
  The acceptance criterion now checks the one-row floor right after the apply and before any resize.
- **F3 (MEDIUM, design).**
  D8's Bound now says `run-shell` format-expands the whole command.
  Every pinned token has each `#` doubled after shell quoting, and `#{q:client_name}` stands as its own unquoted word.
  D11 refers to the same rule.
  A new acceptance row covers a pinned path that contains `#`.
- **F4 (LOW, design).**
  D7 now subtracts the title row only when the readback is exactly `top`.
  `off`, `bottom`, an empty value and a failed readback all mean no title row at row 0.
  The acceptance criterion now names `off` or `bottom`.
- **F5 (LOW, decision).**
  D15 now skips a file that exists but cannot be read, and the acceptance criterion now includes this case.
- **F6 (NIT, decision).**
  D15's temporary file now has a name unique to the run, both in the Decision and in its Bound.
- **F7 (LOW, consistency).**
  D12 now states the revival predicate once, and both the booter's trigger and the revive function's skip use it.
  The entry drops the spawn-order key: the teller sorts the list and also passes the booter's own worktree directory name, and the predecessors are the entries before the matching one.
  `reedengine` sorts nothing.
- **F8 (NIT, consistency).**
  `## Review rounds` now has a round 3 entry and a round 4 entry.

## Disputed

None.

## Deferred

None.
