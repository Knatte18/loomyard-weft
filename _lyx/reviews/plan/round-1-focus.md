---
round: 1
focus:
  - "Fidelity: walk every Decision and Constraint in `_lyx/discussion/decision-record.md` (D1 to D7 and the acceptance criteria) and settle which card carries each; also settle whether any card adds scope the record does not license."
  - "Reachability of refusals in card 1 (`--work` ordering, `--out` checks) and in cards 5 and 6 (`--code`, `--records` refusals): for each prescribed refusal, settle whether the named input reaches it past an earlier check, such as the missing-`--records` default introduced in card 4's `runForkaudit` edit, or the `--work`-before-delete ordering."
  - "Test economy: settle whether `TestForkauditPackCheck`, `TestForkauditTraceCrossCheck` and `TestForkauditPlanSection` each cover behavior that rows of `TestForkauditReport` or `TestForkauditRefusals` already cover, and whether the budget rows in `cmd/lyx/testdata/test-budget.yaml` match the number of new top-level tests per card."
  - "Card order and fixtures: settle whether card 2's fixtures and helpers (`forkFixture`, `placeForks`) and card 4's `runForkaudit` edit leave cards 5 and 6 depending only on earlier cards, and whether card 6's edits to `forkFixture`, `placeForks` and `recordsRepo` respect the re-sign rule."
  - "Verify coverage: settle whether the overview's `## verify:` section runs every package the cards target (`tools/forkaudit`, `cmd/lyx`, `internal/gitkit`) in the order the plan template states, including the integration-tagged tests."
  - "Attack surface: settle whether card 1's `--work` deletion and the card 4 and 5 git reads state in `**Intent:**` what they can delete or touch and what bounds them."
---

Focus is a starting point; the full rubric applies.
