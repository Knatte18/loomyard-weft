## Goal

````markdown
Imported from [issue #260](https://github.com/Knatte18/loomyard/issues/260).

## What happened

A verify gate failed on `TestGateTestMap/NamedPackageIsRebuiltWhenFresh` (`internal/gatecli`, tier 1), on a branch that touched nothing in `internal/gatecli` or `internal/testcover`.

The only mismatch was one extra entry: the fixture test `alpha.TestAlphaTwo` was reported as passing only when run by itself.

```
tier1 (exit 1):
  github.com/Knatte18/loomyard/internal/gatecli TestGateTestMap/NamedPackageIsRebuiltWhenFresh:
    internal/gatecli/testmap_integration_test.go:1124: runTestMap() mismatch (-want +got):
    gatecli.testMapOutcome{
    ... // 1 ignored and 2 identical fields
    TimedOut:    {"alpha.TestAlphaHangs"},
    FailedTests: {"alpha.TestAlphaHangs"},
    - 	PassedAlone: nil,
    + 	PassedAlone: []string{"alpha.TestAlphaTwo"},
    LeftOut:     {},
    Exit:        1,
    ... // 1 ignored field
    }
```

The same subtest passed in the run's other verify logs, and the `TestGateTestMap` run alone through `lym gate test --tier 1` passed.

## Why it is abnormal

A gate goes red on a test the change does not reach, and the agent has to spend a round deciding whether the failure is its own.

## What to do

The likely cause, not confirmed, is load: the test runs with a shortened per-test limit, and a healthy fixture test can exceed it in a full run.
Make the healthy fixture tests insensitive to that limit under load, for example by separating the limit that `TestAlphaHangs` must exceed from the one the other fixture tests run under, or by widening the margin.

Provenance: task `audit-flags-master-friction`, note `burler-code-r1.md`.
````

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-26): No reproduction run on the base

Decision: The parent runs no test on the base. Fix the flake from the code: find why NamedPackageIsRebuiltWhenFresh sees an unexpected PassedAlone entry under load, and make the test or the code deterministic.

Rationale: The failure is a load flake, so a pass on a quiet machine shows nothing; the orch does not run tests.

### Added after Discussion (parent, 2026-11-26): No test seam: the flake is in the test's own time limit

Decision: The red-first gate cannot judge this fix: TestGateTestMap ran its healthy fixture tests under the same 3 s limit its hanging fixture test must exceed, and the fix (00c434749) changes only internal/gatecli/testmap_integration_test.go. On the base the test passes on a quiet machine, so no test can fail there.

Rationale: The defect is a load-dependent timing in test code alone; production code has no bug for a test to catch.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
