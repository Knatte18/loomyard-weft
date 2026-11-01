# Webster forks sweep every retired name, tests included, before editing

A card that removes or renames a name used to leave its fork to find the stale references one gate failure at a time.
In one run, a card that dropped the `loom.plan_*` config keys grepped only non-test files, and stale test fixtures then failed across about ten packages, costing 69 of the fork's 165 turns.

The Webster implementer stencil, `contracts/stencils/webster/webster-body-implementer.md`, now carries a sweep step in each card's step 2.
When a card retires a name, such as a symbol, file, config key or string literal, the fork greps the whole worktree for every retired name in one Bash call before its first hand edit, test files, `testdata/` and other fixtures included.
It reads every hit's span in one message and edits every hit in one message.
For a symbol rename the sweep runs after the hub's rename tool, so it catches what the tool leaves, such as string keys, comments and test data.

## Design decisions

- An edit to a hit outside the card's targets counts as a fix to a failure the card caused, the existing second sanctioned deviation, so the sweep adds no new deviation kind and the scope guard still records each one.
- The step sits beside the quarry-resolve reading rule rather than in the `Rename` sentence, because a card can retire a name without a `Rename` pair, such as by deleting a config key.

## Tests

`TestRenderPrompts` in `internal/websterengine/template_test.go` now pins the sweep's grep and its one-message read and edit, in order after the quarry-resolve line, in the rendered fork prompt.
