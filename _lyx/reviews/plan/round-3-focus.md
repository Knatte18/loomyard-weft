---
round: 3
focus:
  - "Where card 10 adds `Env.PrepareDiscussionSpawn` as a seam `discussionWriteEntry` requires through `requireSeam`: settle whether the card's `**Edit:**` list and `**Intent:**` cover every place that builds a complete `shedrecipe.Env` or enumerates its seams, namely `internal/testkit/envkit/envkit.go` (`FullEnv`, asserted by `NilSeams(FullEnv(t))` in `envkit_test.go`) and the `intentionallyNil` table in `internal/battencli/wire_seams_test.go` (`TestWire_EverySeamFilled`), and whether its `**ImpactSummary:**` names them."
  - "Where card 10's fix touches `FullEnv` or `intentionallyNil`: settle whether the plan still passes the `path-missing`, `bare-symbol-target` and card-order checks of `loom-plan-spec.md`, and whether any card earlier than card 10 already depends on the changed fixture."
  - "Across the plan's `## verify:` section: settle whether every package a card targets after the card 10 change, including `internal/testkit/envkit` and `internal/battencli`, has its tests run by a verify command."
---

Round 3 re-checks the one scope defect around the new required seam and its fixtures.
