---
round: 3
focus:
  - "Footprint of card reflect-under-step: it removes the loomCLI.armedVerb field and its assignment in armAt, but TestArmAt_RecordsTheArmingVerb in internal/loomcli/wiring_test.go (around lines 681-693) reads c.armedVerb. Settle whether the card's Edit list now includes internal/loomcli/wiring_test.go, whether its Intent says what happens to that test, and whether its claim that the field has no other reader and its ImpactSummary are accurate."
  - "Footprint of card halt-notes: it removes shouldReflectFriction, but TestShouldReflectFriction_AwaitingNeverReflects in internal/loomcli/approve_awaiting_test.go (around lines 69-73) also calls it, besides the table test in internal/loomcli/friction_test.go. Settle whether the card's Edit list now includes internal/loomcli/approve_awaiting_test.go, whether its Intent says what happens to that test, and whether its ImpactSummary's claim about who calls shouldReflectFriction is accurate."
  - "For every symbol, field or test that any card removes, grep the repository for remaining references and settle whether each referencing file is in that card's targets."
---

The first two entries are the round-2 BLOCKING defects.
The third checks whether the same kind of defect appears anywhere else in the plan.
