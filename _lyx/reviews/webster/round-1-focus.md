---
round: 1
focus:
  - "Caps counted from settle records (cards 10, 12, 13, 14): does a restarted driver keep the repair cap, the one verify retry and the one hand-over per build, and are the stop key and record reads consistent between the rule table, the loop and batten?"
  - "Driver lock as liveness (cards 15, 16, 19, 20): do start, resume and batten read the same lock path and the same staleness test, and can a halted driverless go child be re-stepped twice for one episode and build?"
  - "Tackler call and consume (cards 7, 8): does the tamper check, the paused-time deadline and the per-stop-key attempt count match the plan, and is the applied mark written after the Admit and never before?"
  - "Unbounded waits and spawns (cards 12 to 15): does every spawn go through a faked seam and log through internal/logger, and is every wait one of the plan's three named waits?"
  - "Per-card mechanical checks for Rename and Delete groups, and new top-level tests against the cmd/lyx/testdata/test-budget.yaml rows: did each group's check run against its own targets, and does any new test duplicate a covering one?"
  - "Seam wiring (cards 21, 22 and the last two commits): are the batten-only InnerRun seams nil-legal in loom's wire, and is every one non-nil where batten wires it?"
---

Seed focus drawn from the plan's Shared Decisions and the cards' riskiest cross-package contracts; the reviewer's base diff review still applies in full.
