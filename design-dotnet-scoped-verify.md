# [.NET] Scoped build and test for C# hubs, and the Models verification wants

`dotnet-scoped-verify` · Task · feature, enhancement, shed, webster, loom, config, dotnet · running · Plan-Burler-Review

## Scope (operator, 2026-10-24)

One loom bundle of notes `dotnet-scoped-verify` and `dotnet-verify-protocol`: verify gates in a .NET hub build and test only what a change touches, plus the remaining wants of the Models verification protocol.

- In scope: the scoped build and test of the first part, Wanted 4 to 6 and the reviewer-rubric repo rules of the second.
- Required 1 and Required 3 went to task `test-tiers`, which has landed; build only what it left out.
- Out of scope: the numerical baseline (Required 2 and its detached-capture addendum), which belongs to note `regression-baseline-verify` and is copied there.

## Scoped build and test (from note `dotnet-scoped-verify`)

### Seen (operator, 2026-10-12)

The operator will use LoomYard for C# development on work machines.
There Millhouse's mill-go often takes about 2 hours for implementation alone, against a median of 27 minutes on this Go codebase.
The machines run Cortex XDR, which scans every file write and process start, and IT allows no scan exclusions.
XDR has slowed only Go tests, which build new binaries; .NET runs through DLLs and has not been slowed, on Linux or Windows 11 (operator, 2026-10-22).
Scoping still pays on its own: every gate tests only what is relevant, with one repo-wide check at the end of each task.

### Want

The verify gates in a .NET hub build and test only what a card or batch touches, as `lyx gate test` scopes Go packages here:

- map changed code to the tests that exercise it, and run only those with `dotnet test --filter`.
  Unlike Go, where a package's tests sit in the package, C# tests live in separate test projects, often one large project for many production projects, so picking test projects alone narrows little (operator, 2026-10-12).
  Candidate maps: test classes that reference the changed symbols (from the index, quarry), naming conventions (`FooTests` for `Foo`), or coverage-based test impact analysis;
- restore once per run, then build and test with `--no-restore` and incrementally;
- keep build processes warm (MSBuild node reuse, build server) so fewer processes start;
- run the whole solution once, at the final gate only.

### Test tiers in the work repository (operator, 2026-10-17)

Operator view, not a decision.
The C# work repository has three tiers of tests: unit tests (small and fast), simulation tests (longer simulations that check the numbers are sane) and regression tests (the baseline, out of scope here).

### Open

- How a hub declares its language toolchain and the commands the gates run, and how much of today's Go-specific gate code this touches.
- Which map picks the tests, and how a missed test is caught: the final full-solution gate, or a card naming its tests under `Uses:`.
- A filtered run still builds the whole test project, so a large shared test project keeps a fixed build cost per gate.
- Measure on the work machine before and after: build and test minutes per run.

## Models verification protocol (from note `dotnet-verify-protocol`, issue #609)

NORCE-DrillingAndWells/Models (C#/.NET 9, NUnit) moves from Millhouse to LoomYard.

### Repo facts the tool has to know

- **Project root is not the git root.** Everything runs from `src/csharp/NORCE.Models/` (it holds `NORCE.Models.sln`, `CLAUDE.md`, `.scratch/`, the task-state dir). Verify commands, scratch paths and baseline paths are relative to that directory, never to the git root.
- **Several task worktrees run at the same time**, often three to five at once, each building and testing. They share one machine.
- **Plain `dotnet test` on the solution is safe:** the two gated projects set `IsTestProject=false` unless their MSBuild property is passed, so the solution run only runs the fast suite.
- Test namespaces are flat (`NORCE.Models.Tests`), so `--filter FullyQualifiedName~X` matches a **class** name, not a folder.

### The three test projects

| Project | Gate | Runtime (quiet machine, 2026-10-14) | When it must run |
|---|---|---:|---|
| `NORCE.Models.Tests` | none | ~2 min, 1537 tests | every implementation attempt (the plan-level verify) |
| `NORCE.Models.SlowTests` | `-p:RunSlowTests=true` | ~2 min, 21 tests | once at finalization, before the PR |
| `NORCE.Models.NumericalBaseline` | `-p:RunBaseline=true` | ~9 s, 8 fixtures | twice: capture before the first change, verify at finalization |

A full solution build is ~30 s.
Under three parallel worktrees the fast suite was observed at 2.5–3.7 min per run.

### Wanted 4: fewer full-suite runs per task

One task (eight fixes, several batches) ran the full fast suite about 15 times, ~35 min, through implementer, fixer and review rounds.
- Let an implementer's in-batch check be **scoped** (`--filter FullyQualifiedName~<Class>`) and keep the full suite to the gate.
- **Skip the gate for a verified tree:** the verified-tree record should also cover the slow tier, keyed on the tree hash, so a step after a passing gate reruns nothing.
- **Doc-only diffs skip all test tiers**: when the diff touches only `*.md` (no `*.cs`, `*.csproj`, `*.sln`), no suite runs.

### Wanted 5: exclusivity for timing work, and only for it

Timing measurements (BenchmarkDotNet runs, before/after tables recorded as deliverables) are invalid if any other worktree is building or testing. Correctness runs are fine in parallel, only slower.
- A container-wide **exclusive lock** a task can take for a timing run, which other tasks' verify runs wait on (and only those, not their agents).
- Never serialize correctness runs.

### Wanted 6: one build, several tiers

The tiers can share one build: `dotnet build NORCE.Models.sln -p:RunSlowTests=true -p:RunBaseline=true`, then each tier with `--no-build`.
This saves ~30 s per extra tier at landing.
If the tool builds once, the gate properties must be passed to the build too, or the gated projects build as non-test projects and discover nothing.

### Repo rules a reviewer rubric should be able to check

Not tool features, but things a configurable review checklist should carry for this repo:
- No test in any of the three projects runs, mirrors or references a `Program*` demonstration (`NORCE.Models.Program`). Check with a grep of the test projects for `Program0NN_`.
- Comparisons against stored output live in `NORCE.Models.NumericalBaseline` alone; an in-process test may byte-compare two arms it builds itself.
- No argument guards (`ThrowIfNull`, range checks) and no try/catch around the repo's own code.
