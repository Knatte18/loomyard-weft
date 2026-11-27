# Flaky gate: TestGateTestMap/NamedPackageIsRebuiltWhenFresh reports an unexpected PassedAlone entry under load

`gatemap-passedalone-flake` · Task · bug, gate · running · Describe

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
