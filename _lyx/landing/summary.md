# A test compiles the C# testdata fixtures with dotnet

Quarry parses the C# fixtures with tree-sitter, which tolerates syntax errors, so a broken fixture still passes the glyph tests.
`TestCSharpFixtures_CompileWithDotnet` in `cmd/lyx/csharpfixtures_integration_test.go` now compiles them with `dotnet build`, so an invalid fixture fails a test.
The change adds only this test; no production code, gate config or fixture changes.

## Design decisions

- The file carries `//go:build integration`, because it spawns `dotnet` and `PATTERN-test-speed` bars spawns from untagged files.
  A machine with .NET checks the fixtures in the gates that run the tier, and a Go-only machine skips the test and stays green.
- The test carries `//lyx:guard`: it reads fixtures in other packages, so the impacted set cannot select it, and the marker makes the round gate run it.
- A `.cs` file is a fixture when any directory on its module-relative path is named `testdata`.
  The scan skips `.git`, `_lyx`, `.lyx`, `_mill`, `.scratch`, `.wiki` and `_raddle`.
- Each fixture directory builds as one class-library project in a `t.TempDir()`, with default compile items off and the files included by absolute path.
  Files in one directory form one compilation unit, as the glyph tests read them, while separate directories may reuse type names, and the repository tree gets no `bin/` or `obj/` output.
- The target framework is `net<major>.0`, with `<major>` read from `dotnet --version`, so no pinned framework can fail on a machine whose SDK cannot target it.
- `dotnet build` runs with `--disable-build-servers`, and the environment opts out of telemetry, the banner and the first-run setup, so no build server outlives the test.
- The test skips when `dotnet` is not on `PATH`, and when `dotnet --version` fails, which a host with the runtime but no SDK does.
  A scan that finds no fixture fails through `scankit.RequireFloor` instead of passing vacuously.
- Compiler warnings do not fail the test; only errors do.

## Risks

- A future fixture meant to be invalid C#, to test quarry's error tolerance, would fail this test and would need an exclusion the test does not carry.
- An SDK older than 7 rejects `--disable-build-servers`.

## To check by hand

- Run `lyx gate test --tier integration ./cmd/lyx` on a machine with .NET to see the test build every fixture directory.
  The `integration` tier was not run on a .NET machine for this description.
- Introduce a type error into a fixture and confirm the test fails naming the directory and the compiler output.
