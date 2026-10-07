---
round: 4
focus:
  - "Notice channels for an awaiting stop that carries a `parent_notice`: D4 suppresses batten's state-changed notice and D6 makes the driver relay the one channel. Settle whether a dead driver or a failed relay send leaves that stop announced by no channel, and whether the decision states what bounds it."
  - "The `since` time in D4: it is taken from the child's newest history entry, while a paused or failed child appends none. Settle whether the 3-minute report wait, the report-time filter and the notice's `since` run from a time that matches when batten saw the state change."
  - "D3's modification-time baseline for the status wait: settle whether the decision states when the baseline is taken relative to the call's status read, so a child write between the two is always seen."
  - "D7's one-invocation window creation with `remain-on-exit off`: settle whether the chained `set-option` has a target that reaches the new window and not the strands' window, and whether the no-live-Selvage fallback keeps the claim that a batten window is never read as the strands' panes."
  - "Cadence of each read in D3's 2 s check (status stat, driver liveness, notice retry, resume retry) and the end of the not-queued retry in D4: settle whether each has a stated cadence and a bound on its log output."
---

Entries name where to look; each restates the sites it depends on and none pre-states a verdict.
