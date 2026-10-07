# Round 1 fixer report

## Fixed

- **F1 (BLOCKING, design).**
  `_lyx/plan/01-shuttle-ttl-config.md` now lists `cmd/lyx/testdata/test-budget.yaml` under `**Edit:**` and its `**Intent:**` raises the `internal/shuttleengine` row from 140 to 141.
  `_lyx/plan/03-claudeengine-prompt-cache-ttl.md` now lists the same file under `**Edit:**` and its `**Intent:**` raises the `internal/shuttleengine/claudeengine` row from 89 to 90.
  Both raises sit in the commit of the test that needs them.
- **F2 (LOW, consistency).**
  Card 1's `template.yaml` bullet no longer has the trailing comment name `5m` and `1h`; it says the values are Claude Code's prompt-cache TTLs, validated when a run starts, and that the refusal names the accepted ones.

`lyx loom validate-plan` exits clean on the edited plan.
`approved:` in `00-overview.md` is untouched and stays `false`.

## Deferred

Nothing.
