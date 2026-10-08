---
round: 1
focus:
  - "Shadow-only sites (cards 13, 14, 15): check that every state-reading call only logs, and that a failed state read leaves each decision, notice and return unchanged."
  - "Card 4 before card 5: check that batten's `LastActivity` ignores `idle_prompt` and other lines `ParseEvents` skips, and that no commit lets the new hooks move batten's quiet clock."
  - "Provider seam: check that `shuttleengine` imports no provider, and that process liveness and the interrupt marker reach it only through `SessionProber` and `ActivityReader`."
  - "Wait loop cursor: check that the signal cursor stops before a trailing unpaired stamp, and that `run.offset`, `ParseEvents`, `EventKind` and every wait-loop verdict are unchanged."
  - "Card 11 background shells: check that a shell whose signal is not `SignalPayload` keeps today's `background_shell_wait_min` expiry, and that a gated run with outputs finishes at once."
  - "Test economy: check each test the diff adds against `cmd/lyx/testdata/test-budget.yaml`, so a new top-level test function raises its package row by the count added in the same card."
  - "Per-card mechanical checks: check that the type-specific checks of each card's groups ran against every group's own targets, including the deny-table and python-deny cards (1, 2)."
---

Focus lists where the plan's shared decisions are most likely to be broken by the diff; it is not a verdict on any of them.
