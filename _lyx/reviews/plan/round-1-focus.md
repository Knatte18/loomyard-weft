---
round: 1
focus:
  - "Card 17 against `contracts/recipes/batten-recipe.yaml` and `contracts/recipes/loom-recipe.yaml`: does the card cover every section of `orch-template-role.md` that the rework finding F1 asks to be checked against the recipes, or only the three it names? Settle whether its claims rows pin the facts F1 lists."
  - "Fidelity to `findings.md` F2, last sentence: it asks to go through the other new tests of the same kind, and cards 18–21 are the plan's answer. Settle whether each card's scope is licensed by F2 and whether any test F2's sweep would reach is left out."
  - "Cards 18, 19 and 21 fold tests into tables and claims rows. Settle whether each folded check keeps what the deleted test asserted, and whether any new claims row or table duplicates behavior an existing test already covers (`PATTERN-test-economy`)."
  - "Card 20 deletes delegation tests and cites the tests that cover the behavior. Settle whether the cited tests exist and cover it, since F2 says to delete only where no existing test already does."
  - "`## verify:` against the packages cards 17–21 target (`contracts/stencils`, `cmd/lyx`, `internal/loomcli`, `internal/orchengine`, `internal/shuttleengine`, `internal/shuttleengine/claudeengine`): does the section run the tests of each, including build-tagged ones?"
  - "Granularity of cards 17–21 and the order of their `**Commit:**` lines: is each card an independently reviewable unit, and does a card's `Uses` or intent depend on a row another card adds (cards 17 and 19 both touch `claims_test.go`)?"
---

Rework generation: `_lyx/loom/rework/round-1/findings.md` (F1 consistency, F2 design) joins the decision record as the answer key.
The plan is four cards of test-only changes plus one stencil card, so the attack-surface and `Custom` questions have little to bite on.
