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

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
