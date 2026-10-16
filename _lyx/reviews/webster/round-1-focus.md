---
round: 1
focus:
  - "internal/boardcli/intake.go, intakeImport: does the success, fold and close-failure output carry exactly slug, issue, labels and dropped (the close-failure error also the close payload), with labels a non-nil list and no entry or body key left?"
  - "internal/boardcli/intake.go, intakeList: does filtering stay identical in both modes, and does each mode print an empty array, not null, for an empty inbox, with the error envelope unchanged?"
  - "internal/boardcli/intake_test.go: do the table-driven list rows reach intakeList through the cobra --full flag under clihelp.RunRootCtx, and do the import tests read the stored entry through b.GetTask as card 2 states?"
  - "Test economy: does any new list or import row duplicate another row's coverage, or does a behavior the diff adds have no row?"
  - "contracts/stencils/orch/orch-template-role.md: does the inbox check keep the card's bounds (list only by the orch, empty ends, error envelope not a fork, no second fork, fork outcomes and one-line report), and is the last sentence of Where a finding goes replaced without dropping a rule the card did not ask to change?"
  - "plugins/ly/skills/board/SKILL.md and the stencil: do both name only commands the agents' settings allow and ask for no skill load, and does the SKILL.md text match the new list and import output?"
  - "internal/boardcli/doc.go: does each output change appear in the paragraph of the card that makes it, with the keys the code prints?"
---

Seed focus from the three cards of the plan.
No round has been judged yet.
