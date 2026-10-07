# Round 4 fixer report

Verdict was APPROVED with four findings; all four are fixed.
`approved:` in `00-overview.md` is untouched.

## Changed

- **F1 (MEDIUM, scope)** — `_lyx/plan/09-mark-guards.md`: added `contracts/stencils/parentdirective_test.go` and `contracts/stencils/roleskills_test.go` to the `**Edit:**` list, and replaced "the two stencil scans the decision names" with a sentence naming `claims_test.go` as the decision's and the stencil scans that enforce `PATTERN-role-skills-typed` and `PATTERN-parent-directive` as reached through the `(test)` entries.
- **F2 (LOW, scope)** — `_lyx/plan/09-mark-guards.md`: added `cmd/testtiming/main.go` to the `**Edit:**` list and a sentence in the Intent that its package comment states the stacking rule beside the placement and abort rules.
- **F3 (LOW, consistency)** — `_lyx/plan/11-tmux-tier-green.md`: cut "tmux 3.6", the non-ancestor revision clause and "the failure did not reproduce when the plan was written"; the paragraph now ends on "The test is now the `CrossClientControl` step of `TestSmokeDotFill`."
- **F4 (NIT, consistency)** — `_lyx/plan/00-overview.md`: the Card Index line for card 11 now says a failure whose cause sits in the `reedcli` smoke test files is fixed there and any other stops the run before card 12.

## Deferred

None.
