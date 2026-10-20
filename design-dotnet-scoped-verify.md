# [.NET 003] Scoped build and test for C# hubs

`dotnet-scoped-verify` · Note · feature, shed, webster, dotnet

## Seen (operator, 2026-10-12)

The operator will use LoomYard for C# development on a Windows work machine.
There Millhouse's mill-go often takes about 2 hours for implementation alone, against a median of 27 minutes on this Go codebase.
The machine runs Cortex XDR, which scans every file write and process start, and IT allows no scan exclusions, so every build and test run is slow.
On that machine, narrowly scoped testing is what decides how long a run takes.

Update (operator, 2026-10-20): the operator will also develop Models on OSL-1046, an Ubuntu 24.04 work laptop with Cortex XDR; note `dotnet-bench-osl1046` measures it there first.

## Want

The verify gates in a .NET hub build and test only what a card or batch touches, as `lyx gate test` scopes Go packages here:

- map changed code to the tests that exercise it, and run only those with `dotnet test --filter`.
  Unlike Go, where a package's tests sit in the package, C# tests live in separate test projects, often one large project for many production projects, so picking test projects alone narrows little (operator, 2026-10-12).
  Candidate maps: test classes that reference the changed symbols (from the index, quarry), naming conventions (`FooTests` for `Foo`), or coverage-based test impact analysis;
- restore once per run, then build and test with `--no-restore` and incrementally;
- keep build processes warm (MSBuild node reuse, build server) so fewer processes start;
- run the whole solution once, at the final gate only.

## Test tiers in the work repository (operator, 2026-10-17)

Operator view, not a decision.
The C# work repository has three tiers of tests, and the verify gates must handle each:

1. Unit tests: small and fast.
2. Simulation tests: longer simulations of the drilling system that check the numbers are sane, such as no negative pressure and no values far out of range.
3. Regression tests: a suite run before the implementation starts writes profiles, and the same suite run after the implementation compares against them to catch drift.
   Many tasks in that repository are pure refactorings that must not change behaviour, so this tier is what proves them.

Orch idea, not a decision: the regression tier needs a baseline step at the start of a run, on the parent's code before any card lands, and a compare step at the final gate, so the profiles become run state like the plan.

## Open

- How a hub declares its language toolchain and the commands the gates run, and how much of today's Go-specific gate code this touches.
- Which map picks the tests, and how a missed test is caught: the final full-solution gate, or a card naming its tests under `Uses:`.
- A filtered run still builds the whole test project, so a large shared test project keeps a fixed build cost per gate.
- Measure on the work machine before and after: build and test minutes per run.
