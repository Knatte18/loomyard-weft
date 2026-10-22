# Decision record — csharp-fixtures-compile

## Goal

Prove that every C# test fixture under a `testdata/` directory is valid C#, by compiling it with `dotnet build`, so a broken fixture fails a test instead of passing quarry's error-tolerant tree-sitter parse.

## Scope

- One new `integration`-tier test that finds every `.cs` file under any `testdata/` directory of the module and compiles it with `dotnet build` in a throwaway project.
- The test skips when `dotnet` is not on `PATH`.
- No production code, gate config or fixture changes.

## Decisions

### The test lives in the `integration` tier

Decision: the test file carries `//go:build integration`.
Rationale: it spawns `dotnet`, and `PATTERN-test-speed` bars spawns from untagged files; `tier1` runs in the batch, round, final and landing gates, so a machine with .NET checks the fixtures on every such gate, and a Go-only machine skips the test and stays green.

### The test sits in `cmd/lyx` and carries `//lyx:guard`

Decision: the test goes in `cmd/lyx/csharpfixtures_integration_test.go` beside the other repository scans, marked `//lyx:guard`.
Rationale: it reads fixtures in packages other than its own, so the impacted set cannot select it; the marker makes the round gate run it whenever it runs.

### One throwaway project per fixture directory

Decision: the test groups the found `.cs` files by directory and builds each directory as one class-library project in a `t.TempDir()`, whose `.csproj` includes the fixture files by absolute path, with default compile items off.
Rationale: fixtures in one directory form one compilation unit, as the glyph tests read them, while separate directories may reuse type names; the repository tree gets no `bin/` or `obj/` output.

### The target framework follows the installed SDK

Decision: the project's target framework is `net<major>.0`, with `<major>` read from `dotnet --version`.
Rationale: a pinned framework fails on a machine whose SDK cannot target it; the fixtures use no framework-specific API.

### The build leaves no process or telemetry behind

Decision: `dotnet build` runs with `--disable-build-servers`, and the environment sets `DOTNET_CLI_TELEMETRY_OPTOUT`, `DOTNET_NOLOGO` and `DOTNET_SKIP_FIRST_TIME_EXPERIENCE`.
Rationale: MSBuild node reuse and the shared compiler server otherwise outlive the test.

## Constraints

- `PATTERN-test-speed`: a spawn only in a tier-tagged file, `t.Parallel` by default.
- `PATTERN-test-tiers`: the test runs only through `lyx gate test`.
- `PATTERN-trimpath-safe-root`: the module root comes from `scankit.Root`, never a `runtime.Caller` path.
- `PATTERN-lyx-temp-root`: the throwaway project lives in `t.TempDir()`.
- `PATTERN-comment-line-breaks`: one sentence per comment line.

## Auto-mode assumptions

- A darn run holds no interview; every call here is the writer's self-pick.
- A `.cs` file counts as a fixture when any directory on its module-relative path is named `testdata`.
- Compiler warnings do not fail the test; only errors mean the fixture is invalid C#.

## Open risks

- A future fixture meant to be invalid C#, to test quarry's error tolerance, would fail this test; such a fixture would need an exclusion the test does not carry yet.
- An SDK older than 7 rejects `--disable-build-servers`; this repository's .NET machines run a newer SDK.

## Acceptance criteria

- With `dotnet` on `PATH`, the test builds every fixture directory and passes on the current fixtures.
- A syntax or type error in a fixture fails the test, naming the directory and the compiler output.
- Without `dotnet` on `PATH`, the test skips.
- A scan that finds no `.cs` fixture fails rather than passing vacuously.
