---
round: 1
focus:
  - "Fidelity: check each of D1-D18 and every Constraint in the decision record against the cards; look for a decision no card carries, especially D7, D8, D13 (queue cap, idle-gated delivery) and D17, D18."
  - "Fidelity: check each card's scope against the decision record; look for scope the record does not license, in cards 7-10 (notice queue, batten wiring, stencils) and card 12 (smoke)."
  - "Verify coverage: settle whether the `## verify:` section runs the tests of every package any card targets, including `internal/proc`, `internal/shuttleengine/claudeengine` and the stencil/doc-only targets; `-tags smoke` compiled by `go vet` is not a finding."
  - "Attack-surface: for the guards cards 3 and 11 change (`permission_mode` narrowed, PID-reuse adopt let through) and the new notice queue and `/compact` verb in cards 4, 5 and 7, settle whether `**Intent:**` states what is let through and what bounds it."
  - "Granularity: settle whether cards 4/5 and 8/9 split independently testable units, or whether any card bundles unrelated units (card 7 pairs the notice queue with a PATTERN entry)."
  - "Consistency: check that the Shared Decisions' `Applies to` card lists match the cards that carry each decision, and that card Intents agree on the names they share (`ContextReading`, `IdleProbe`, `QueueNotice`, `PrimePaths`, `State.Stuck`)."
---

Seed focus only; every entry names where to look and what to settle.
