## Goal

```markdown
## Aim

`docs/research/speed/test-time-measure-2026-11-27/report.md` measured `internal/gatecli` at 49 to 56 s of tier1 test time, mostly two long sequences on one fixture:
`TestGateTest_Scopes` (54 s) and `TestGateTest_Scenario` (45 s).

## Work

- Split each into independent subtests or tests with a fixture each, so they run in parallel.
- Move scope computation cases that need no subprocess to tier0.
- Keep every assertion; drop none.
- Leave `TestGateRedFirst_Dotnet` and `TestGateTest_DotnetFixture` alone: they run only under upkeep.
- Measure the package's tier1 time before and after, twice each, and put both in the change description.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-27): Writer measures gatecli tier1 time itself

Decision: The writer may run lym gate test --gate darn-write --wait --no-carry --tier 1 ./internal/gatecli -- -count=1 -v in the foreground, twice on the untouched base and twice after its commits, and put both pairs with the load average at each start in the change description. It runs no other gate test beyond its normal gates.

Rationale: The task exists to cut this package's test time; only the writer's worktree holds the before and after trees, and the orch runs no tests.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
