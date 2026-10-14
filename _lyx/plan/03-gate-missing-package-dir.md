# Card 3 — gate-missing-package-dir

**Create:**
- `plan:internal/gatecli#missingPackagesError` -> `func missingPackagesError(dir string, packages []string) string`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#newCommand`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:** D4: `lyx gate test` refuses a relative package argument that names no directory.

`missingPackagesError` lives in `internal/gatecli/cli.go` beside `refusePackages`.
For each package argument spelled relative (`.`, `..`, `./…`, `../…`) and not ending in `.go`, it removes any trailing `/...` (so `./...` checks `dir` itself), joins the rest onto `dir` with `filepath.Join` and `filepath.FromSlash`, and collects the argument when `os.Stat` fails or the result is not a directory.
It returns "" when none is missing; otherwise one JSON-error text naming every missing argument in order, for example `gate test: package argument ./nope names no directory under <dir>`.
An argument ending in `_test` whose trimmed directory (the `_test` suffix of its last segment removed) exists gets its own clause: it is an external test package whose tests live in that directory, run `./internal/x` instead.
The text ends with the way forward `re-run the same command with each package argument naming an existing package directory`, and for a `_test` argument names the corrected command line.
Import paths, absolute paths, patterns other than the relative `/...` form and `.go` file arguments are never collected, so they reach go test unchanged.

`runTest` calls it right after `absoluteDir` resolves the directory (the `-C` value or the seam cwd from `lyxcwd.CwdFrom`, never `os.Getwd`), before `catchTerminatingSignals`, `resolveHub`, any slot path (inherited, slotted or outside every hub) and the prebuild, and returns `output.Err(out, text)` (exit 1) when the text is not empty, so a refused request runs no package and waits for no slot.
`newCommand`'s `Long` gains one sentence: a relative package argument that names no directory is refused before any slot is taken.

What the refusal can now stop: a hand-written gate line with one wrong argument, which before ran the other packages, is now refused whole; the bound is that only a relative spelling, checked against the disk, is refused.

Tests:
- `TestGateTest_RefusesBeforeResolvingTheHub` gains a `dirs` field (directories created under the row's own `t.TempDir()` cwd before the call), a `wantAbsent` field (a substring the error must not carry), and rows: `test ./internal/x_test` with `internal/x` created, whose error holds `./internal/x` and `external test package`; `test ./nope ./internal/x` with `internal/x` created, whose error holds `./nope` and not `./internal/x`; `test -C sub ./nope` with `sub` created, whose error holds `./nope`; `test ./nope/...` whose error holds `./nope/...`.
  Each row is outside every hub, so the refusal is the only route to exit 1 with a JSON envelope before any go run.
- `TestGateTest_Scenario`: every existing subtest that passes a relative package argument (`./pkg`, `./held`, `./nested`, `./enclosed`) now needs that directory, so the test creates `pkg`, `held`, `nested` and `enclosed` under `prime` before its first subtest, and creates `pkg` under each `t.TempDir()` cwd a subtest passes `./pkg` from; the `./...` subtests are unaffected.
  The pre-created `nested` is the same `nestedModule` directory the nested-module subtests later fill with a `go.mod` through `os.MkdirAll`, which tolerates it existing, so no second setup is added.
  Two new subtests in its sequence: "a relative argument naming no directory refuses before the slot", running `test --tags integration ./pkg_test` from `prime`, which exits 1 with a JSON error, leaves no stand-in record at all (so neither the prebuild nor go test ran), and then `requireFree`, pinning that the refusal precedes the slot and the prebuild inside a hub; the `_test` hint wording is pinned by the unit row, not here; and "import paths, patterns and .go files reach go test", running `test ./pkg/... example.com/m/x ./pkg/a_test.go` from `prime`, whose stand-in record's args end with exactly those three arguments.

Docs in the same commit:
- `internal/gatecli/doc.go`'s Verb section states the refusal, its bound, and that it precedes hub resolution, the slot and the prebuild; its Exit codes section is unchanged (1 for a refusal).
- `contracts/specs/refusal-spec.md`'s `## gate` table gains a row after `-C directory missing`: `package directory missing` | a relative package argument, its trailing `/...` removed, names no existing directory under the run's directory; an argument ending in `_test` whose trimmed directory exists is named as an external test package with its corrected argument | correctness halt | re-run the same command with corrected package arguments.

**ImpactSummary:** `runTest` gains one refusal ahead of hub resolution; every caller passing an existing relative directory, an import path, a pattern or a `.go` file runs as before, and the integration scenario's fixture directories are created to keep its subtests on the run path.

**Commit:** `3: lyx gate test refuses a relative package argument that names no directory`
