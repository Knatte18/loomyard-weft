# Card 13 — Plan-Write greps for reworded messages

**Edit:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/claims_test.go`

**Intent:**
D2: `loom-template-plan.md`'s card rules gain one rule:
a card that changes a user-visible message or error text greps the repository for the old text, and every test asserting it joins that card's targets.
Stale exact-string assertions then fail on the card that reworded them, under that card's own gate (card 12), not at the plan-level gate.

Test: `claims_test.go` pins the rule's wording in `loom-template-plan.md`.

**ImpactSummary:** no production code reads the rule; only plans Plan-Write writes from now on change, a message-rewording card gaining the asserting tests as targets, and existing plans are unaffected.
