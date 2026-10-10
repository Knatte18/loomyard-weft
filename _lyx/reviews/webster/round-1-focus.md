---
round: 1
focus:
  - "Slot lifetime in internal/gateslot and every caller (cards 1, 4-9): is each acquired slot released on every path, including error, timeout and panic, and is the pool limit read at each acquire rather than captured at wiring time?"
  - "Nil pool handling at every Go-side site (cards 4-9): does a nil *gateslot.Pool run unslotted with the parent environment, and does any hub-mode wiring pass hubgeom.GateSlots instead of nil?"
  - "Per-card mechanical checks: for each card with Rename or Delete groups, did the type-specific check run against every group's own targets, not only the first label's?"
  - "Child lifetime code in lyx gate test (card 10): does it stay in new internal/gatecli files, with no change to internal/proc or internal/verifyrun's process-group files?"
  - "Tier placement (cards 14, 15, 21): does the card gate run only tier 1 over the card's own packages, the batch gate run tier 2 once per batch, and the round gate run tier 3 only while a Publish failure record is present?"
  - "Publish failure record (cards 19, 20, 22): does Publish write the record on failure and clear it on success, and does the rubric render only a checked record?"
  - "Hub-wide config write refusal and landing (cards 17, 18): are writes refused from pairs and pair-spawned sessions on every path, and is landing seeded from the prime?"
  - "Test economy: do new tests duplicate coverage by an existing test or a public-surface test, and does any added behavior lack coverage?"
---

Focus entries name where to look and which question to settle; none pre-states a verdict.
