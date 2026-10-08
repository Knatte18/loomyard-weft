---
round: 3
focus:
  - "Pin arithmetic under `pane-border-status top`: in `decision-record.md` D7 and its acceptance criterion, does the stated pin value for the pane at window row 0 (issued by `resizePinHookArgvs` through `render.FixedHeightPins`, where `resize-pane -y` sizes pane content, not the cell) yield the intended per-cell height after a `window-resized` event, and does the criterion check per-cell heights?"
  - "Color validation channel: in D13 and its acceptance criterion, is the point where an out-of-palette segment color is refused consistent with where reed validates `mouse` and `watchdog` (`ensureServerAndSessionLocked` at boot), and does every other `reedengine.LoadConfig` consumer (`down`, `attach`, `status`, `pairteardown`, the watchdog) fail safe rather than block?"
  - "Ordering bound: in D12, where does the spawn-order guarantee stop for a booter whose later-position sessions are already live, or whose predecessor's nested up failed, and do the Bound, Open risks and the acceptance criterion agree on that stop?"
  - "Lock sequence and teller input: does D12 state where the trigger is read relative to `Up`'s op lock, where the lock is released and re-taken, what is re-checked, and what the teller supplies per entry so `reedengine` runs another worktree's up without deriving a path (`PATTERN-told-geometry`)?"
  - "Atomic write: do D15's temp-file-and-rename write, its read-only promise and its acceptance criterion describe one behavior, including the mode kept on the replaced file?"
---

Focus entries name where to look; the reviewer still reads the whole artifact pair against the rubric.
