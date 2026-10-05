# Round 3 fixer report

## Changed

All edits in `_lyx/discussion/decision-record.md`.

- F1: D6 Bound now scopes the `tmux` run to this task's own plan verify and says other verifies follow the D10 text, otherwise `tmux` is run by hand; the matching acceptance criterion says "this task's plan verify".
- F2: D6 gains a rule that each tag compiles on its own, with shared helpers in an untagged file or a kit; the acceptance criteria add per-tag `go vet` runs.
- F3: the `PATTERN-test-isolation` constraint states that the `!integration`/`integration` `TestMain` pairs stay paired, call `tmuxkit.Main`, and are checked under every tag combination.

## Deferred

None.
