# Card 9 — fix-cap

**Create:**
- `plan:internal/landingshed#FixCap` -> `type FixCap struct`
- `plan:internal/landingshed#FixCap.Exceeded` -> `func (c FixCap) Exceeded(change fabricengine.LineChange) string`
- `plan:internal/landingshed#TestFixCap_Exceeded` -> `func TestFixCap_Exceeded(t *testing.T)`

**Intent:** Judge a fixer change against D10's size and kind caps, the bound on what an unreviewed fix can let through.
`FixCap` holds `MaxLines`, `landing.yaml`'s `finalize_fix_max_lines`, and `IsTestFile`, the told matcher of the run's language (card 3's `planparser.IsTestFile` for a loom plan's language, `planparser.IsAnyLanguageTestFile` for darn).
`Exceeded` returns a reason naming what put the change over, or "" when it is within the caps:
a change whose `Lines` exceed `MaxLines`;
whatever its line count, a change touching a path `IsTestFile` matches, a path with a `testdata` segment, a gate or build configuration file (`go.mod`, `go.sum`, `go.work`, `Makefile`, a `.csproj`, `.sln`, `.props` or `.targets` file, `Directory.Build.*`, a path under `.github/`, or a file named `gate.yaml`), or an added line that skips a test (`t.Skip`, `t.SkipNow`, `[Ignore`, `Skip =`) or adds a build constraint (`//go:build`, `// +build`, `#if`).
The reason names the first offending path or line, so the halt that carries it names the fixer's change.

The table-driven test pins a change of 41 lines and one of 40, a 3-line `_test.go` edit under a Go matcher, a `FooTests.cs` edit under a C# matcher, any file under the over-covering `none` matcher, a `testdata` file, one row per gate or build configuration name (`go.mod`, `go.sum`, `go.work`, `Makefile`, a `.csproj`, `.sln`, `.props` and `.targets` file, `Directory.Build.props`, a path under `.github/` and `gate.yaml`), one row per added skip or constraint line (`t.Skip(`, `t.SkipNow(`, `[Ignore`, `Skip =`, `//go:build`, `// +build` and `#if`), each with its reason, and a small source-only change within the caps.
