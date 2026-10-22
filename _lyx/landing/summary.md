# A test compiles the C# testdata fixtures with dotnet, skipped without it

Quarry parses C# fixtures with tree-sitter, which tolerates syntax errors, so a broken fixture such as `internal/planglyph/testdata/glyphchain/csharp/Renderer.cs` would still pass the glyph tests.
`TestCSharpFixtures_CompileWithDotnet` in `cmd/lyx/csharpfixtures_integration_test.go` now proves each fixture is valid C#.

## What it does

- Scans the module for `.cs` files under any `testdata/` directory, skipping `.git`, `_lyx`, `.lyx` and `.scratch`.
- Builds each fixture directory as one throwaway class library in `t.TempDir()`, one parallel subtest per directory.
  The `.csproj` includes the fixtures by absolute path with default compile items off, so the repository gets no `bin/` or `obj/`.
- Fails on a compiler error, printing the directory and `dotnet build`'s output; warnings pass.
- Skips when `dotnet` is not on `PATH`, so a machine without .NET stays green.
- Fails when the scan finds no fixture, so it never passes vacuously.

## Design decisions

- **Tier.** The file is `//go:build integration`: it spawns `dotnet`, which untagged files may not do, and `tier1` runs in the batch, round, final and landing gates.
  No new tag or `gate.yaml` tier: the skip already keeps Go-only machines green.
- **Placement and guard marker.** It sits in `cmd/lyx` with the other repository scans and carries `//lyx:guard`, because it reads fixtures in other packages that the impacted set cannot connect to it.
- **Target framework.** `net<major>.0`, with the major version read from `dotnet --version`, so the test follows whatever SDK the machine has.
- **No leftover processes.** `dotnet build` runs with `--disable-build-servers`, so MSBuild nodes and the compiler server exit with the build.
  The environment turns off telemetry, the banner and the first-run setup.
  The flag needs SDK 7 or later.
- **No fixture exclusion list.** A future fixture that is meant to be invalid C#, to exercise quarry's error tolerance, would need one added.

## Verification

With SDK 10.0.113 the test passes on the current fixture.
With a type error put into `Renderer.cs` it fails, printing `error CS0246` for that file.
