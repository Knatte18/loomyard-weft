## Goal

```markdown
## Seen (`docs/research/drift-whole-review-2026-11-04.md`, faults 1 and 3)

- `lym drift accept <test>` checks nothing about its test name: the name goes straight into the scope record, and an accepted name joins the compare's skip list (`SkipList`).
  A typo, or a test that never drifted, hides that test from every later compare.
  The same holds for the `--skip` names of `lym drift scope`.
- Drift-Verify captures the accepted profile at the end of its pass (`keepAcceptedProfile`), after the verify agent's fixes, while the agent's later compares skip accepted tests.
  An agent that accepts test X and then edits code that moves X again leaves a profile that records the moved output as the accepted one.

## Wanted (operator, 2026-11-04)

Two interim repairs ahead of the Drift rework the report proposes; nothing else of Drift changes.

1. `lym drift accept` and `lym drift scope --skip` refuse a test name that is not a file under the run's baseline profile directory, naming the name and the directory in the refusal.
2. The accepted profile records each accepted test's output as it was when the test was accepted, not the output at the end of Drift-Verify's pass.
   A later change that moves an accepted test's output is no longer taken into the accepted profile unseen.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-04): Drop the --skip name check; keep the accept check

Decision: Wanted item 1 covers lym drift accept only. lym drift scope --skip stays unchecked: it runs at Drift-Baseline before any capture, and a capture writes no file for a skipped test, so a check against the baseline directory would refuse every partial scope. The capture harness already refuses an unknown skip name, and the partial scope goes in the Drift rework.

Rationale: Option (a) from the writer's question #647: a check that cannot hold as written is dropped rather than rebuilt for a scope the rework removes.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
