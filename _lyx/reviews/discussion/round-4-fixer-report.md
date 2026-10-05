# Round 4 fixer report

## Changed

All edits are in `_lyx/discussion/decision-record.md`.

- F1: D6's closing sentence and the matching acceptance criterion now say `integration` is run by the standing gates, `tmux` by this task's plan verify and by hand until `test-tiers`, and `llm` is compile-checked only.
  The D6 sentence also names `go vet -tags tmux ./...` as the compile check for `tmux` files.
- F2: the test-isolation constraint now says the `!integration` files keep their constraint and are the `TestMain` under `tmux`-only and `llm`-only builds.
  It also states the check's tag sets (untagged, each tag alone, the combined set) and that exactly one `TestMain` compiles under each set on each platform.

## Deferred

Nothing deferred.
