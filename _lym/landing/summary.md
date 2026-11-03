# Drift-Baseline compares intended drift instead of skipping it

The Drift-Baseline stencil (`contracts/stencils/drift/drift-template-baseline.md`) told the agent to pick scope `off` for a task that changes behavior on purpose, and scope `partial` with `--skip` for every test such a task drifts.
A skipped test is never compared, so the intended change was never accepted with a reason, and any regression in the same tests went unseen.
On the `cs` hub, a report-format change skipped all six regression cases this way, although each case renders the report on one line among many raw lines that must not change.

Drift already handles intended drift: the compare reports it, and the Drift-Verify agent accepts each drifted test with `lym drift accept <test>` and a reason, which the parent rules on under scope `full`.
The stencil now routes a task there.

## What changes

- Step 2's scope list:
  - `off` is only for a task the suite cannot reach.
  - `full` is for every task the suite reaches, a refactoring or an intended behavior change alike.
  - `partial` with `--skip` is only for a test the task makes meaningless or nondeterministic.
  - A closing line forbids skipping a test because its output changes on purpose.
- Step 4's report asks for each expected drifting test, the lines or values that will change and why, and states that every other line of it must stay as it is, so Drift-Verify accepts the intended drift and flags anything else.
  It also asks why each skipped test is skipped.

## Design decisions

- No Go code changes: `driftengine` and the `lym drift scope` verb already accept every scope and skip list the new text asks for, and their docs describe mechanics, not when to choose a scope.
- The configured `drift.scope` path, which records a scope with no agent, is untouched.
