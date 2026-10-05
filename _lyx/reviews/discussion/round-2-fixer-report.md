# Round 2 fixer report

All edits are in `_lyx/discussion/decision-record.md`.

- F1: D6 Bound now says `llm` files are compile-checked by `go vet -tags llm ./...` and run by hand, that leaving automatic runs is intended, and that no file may sit under a tag neither run nor compiled.
  The matching acceptance criterion is narrowed to files retagged into `integration` or `tmux` and gains the `llm` compile check.
- F2: D6 Bound says `cardGateCommand` is unchanged, card gates lose the `tmux` tier until `test-tiers`, and the plan verify runs `-tags tmux`.
- F3: D2 states the static-scan rule that recognises a spawning test, over-approximating so errors fall into "no coverage".
- F4: Constraints name `PATTERN-test-speed` and `PATTERN-test-isolation` instead of the retired IDs.

## Deferred

None.
