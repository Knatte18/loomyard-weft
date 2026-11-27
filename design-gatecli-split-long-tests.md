# Split the long gatecli tier1 scenario tests

`gatecli-split-long-tests` · Task · enhancement, gate, testkit · running · Code-Burler-Fix

## Aim

`docs/research/speed/test-time-measure-2026-11-27/report.md` measured `internal/gatecli` at 49 to 56 s of tier1 test time, mostly two long sequences on one fixture:
`TestGateTest_Scopes` (54 s) and `TestGateTest_Scenario` (45 s).

## Work

- Split each into independent subtests or tests with a fixture each, so they run in parallel.
- Move scope computation cases that need no subprocess to tier0.
- Keep every assertion; drop none.
- Leave `TestGateRedFirst_Dotnet` and `TestGateTest_DotnetFixture` alone: they run only under upkeep.
- Measure the package's tier1 time before and after, twice each, and put both in the change description.
