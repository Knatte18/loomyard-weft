---
round: 3
focus:
  - "Card 8 renames `RunDeps.Engine` to `Engines`: check every reader of the old field, including `internal/shedrecipe` (`websterEntry`, its test field lists), `internal/battencli`'s `intentionallyNil` and `internal/testkit/envkit`'s reflection walk, is a target of card 8 or an earlier card, so the tree compiles and its tests pass after card 8."
  - "Card 10 adds the func field `RunDeps.InstructionFiles`: check `envkit`'s `nilLegal`, battencli's `intentionallyNil` and every constructor of `RunDeps` (`loomCLI.wire`, `websterCLI.runDeps`) are covered by a card at or before card 10, and that a test reaches each wiring."
  - "Cards 10 and 11 prescribe refusal rows and way-forward text for `lyx webster`, `lyx orch start` and `lyx burler run`: for each caller, settle whether the step named is one that caller can take, and whether a row's trigger is reachable past card 9's entry gate."
  - "Any card inserted or renumbered this round: check the Card Index, cross-card references in prose (card numbers cited inside other cards' Intents) and the `## verify:` section still agree."
---

The round 2 fixes are new text no fresh reviewer has seen, so these entries point at the sites they touched.
