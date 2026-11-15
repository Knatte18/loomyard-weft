# Let a gated darn writer rerun a reported failing test before diagnosing

A darn writer whose verify gate has a cold fixer was told to run no `lym gate test` at all.
A task that starts from a test failure seen on an older base therefore could not check whether the failure still reproduces,
and the writer spent its session tracing a failure that the current base no longer has (issue #119).

## What changes

- `contracts/stencils/darn/darn-fragment-make-change-gated.md`: Step 4's item 3 now reads "Run no `lym gate test` yourself, bar one run".
  When the task starts from a reported test failure, the writer runs that test once before diagnosing it,
  scoped to its package with `-- -run <TestName>` under the gate rule from its system prompt.
  A pass means the failure no longer reproduces, and the writer treats the entry as one that landed work overtook, which Step 3 already settles.
  The subagent rule is unchanged: subagents still run no gate.
- `internal/darnengine` and `contracts/stencils`: the doc comments that described the gated step as "no gate" name the one allowed run.
- Tests: `TestDarnSpec` asserts the reproduce-run line in the gated prompt and its absence in the ungated one.
  The wiring test's pinned Step 3 line follows the new wording.

## Design decisions

- The writer runs the test itself rather than asking its parent for the run.
  The issue offered both; asking costs a round-trip through the parent on every such task, and nothing outside the prompt refuses a writer's `lym gate test`.
- Only the gated fragment changes.
  The ungated writer already runs the gate itself.
- A pass routes through Step 3's existing outdated-entry path, so the parent decides what the task becomes, and the prompt adds no new stop mechanism.
