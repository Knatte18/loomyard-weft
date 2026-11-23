## Goal

```markdown
Imported from [issue #187](https://github.com/Knatte18/loomyard/issues/187).

Seen: during a run's PR-Review row, lym loom pr-findings failed with: NewParentAlign: Deps.OpenFabric must not be nil.

Workaround: the PR findings were sent to the PR-Review agent by creel mail instead.

Expected: pr-findings builds its parent-align dependencies itself and records the findings.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): Close as overtaken only after a darn-path test

Decision: The orch runs no tests. Add a tier-1 test in internal/loomcli that drives lym loom pr-findings on a darn run through the real cobra tree, beside TestPRFindingsCLI_SendsBackOnATapestryRun. If it passes on your base, the entry is overtaken by 6fa7f1740: land the test and say so in the change description. If it fails, fix the wiring on the darn path.

Rationale: Issue #187 came from an overnight run whose binary cannot be traced now; the installed lym (cce041999, 2026-11-23 08:00) contains 6fa7f1740, but only a tapestry run is pinned by a test.

### Added after Discussion (parent, 2026-11-23): No test seam: the bug was fixed on the base by 6fa7f1740

Decision: TestPRFindingsCLI_SendsBackBlockingFindings passes on the base for both the tapestry and darn rows, since 6fa7f1740 wires the landing seams before pr-findings builds its shed. The entry lands as overtaken, with the test as a guard.

Rationale: No test can fail on a base that already holds the fix.

### Added after Discussion (parent, 2026-11-23): Accept round 1: F1 fixed

Decision: Accept the code review at round 1. F1 (findings-path-built-twice) is fixed in b18c1caa4: blockingFindingsRun returns the findings path, built once, and the subtest no longer rebuilds it.

Rationale: A mechanical test fix, visible in the commit and passing its gates; another round would only confirm it.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
