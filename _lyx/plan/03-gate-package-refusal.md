# Card 3 — gate-package-refusal

**Create:**
- `plan:internal/gatecli#refuseMissingPackages` -> `func refuseMissingPackages(out io.Writer, dir string, packages []string) (int, bool)`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:** D4: `lyx gate test` refuses a relative package argument that names no directory, before any slot or prebuild.

`refuseMissingPackages` lives in `internal/gatecli/cli.go` beside `refusePackages`.
For each package argument spelled `.`, `..`, `./…` or `../…` and not ending in `.go`, it removes any trailing `/...`, joins the rest onto `dir` and checks with `os.Stat` that it is an existing directory.
When any is not, it writes one JSON error through `output.Err` naming every such argument as typed, and returns exit 1 and true; otherwise it writes nothing and returns false.
For an argument ending in `_test` whose trimmed directory exists, the error says it is an external test package whose tests live in that directory and names the corrected argument (`./internal/x` for `./internal/x_test`).
The way forward is to re-run the same command with the corrected package arguments.
Bound: import paths, absolute paths, `all` and `.go` file arguments pass to go test unchanged; a relative pattern is checked on its directory, so `./...` and `./internal/...` pass when that directory exists; a refused request runs no package.

`runTest` calls it right after `absoluteDir`, so the base directory is the `-C` value made absolute against the seam cwd, or the seam cwd itself, never `os.Getwd` (`PATTERN-cwd-resolution`), and the check runs before `catchTerminatingSignals`, `resolveHub`, every slot path (inherited, slotted, outside every hub) and `prebuildLyx`.

Tests:
- `TestGateTest_RefusesBeforeResolvingTheHub` reaches the refusal with no hub, since it precedes hub resolution.
  Its table gains a `setup` of directories created under the cwd and a `wantAbsent` substring, and rows: `./internal/x_test` with `internal/x` present refuses naming `./internal/x` and "external test package"; `./nope ./internal/x` with `internal/x` present refuses naming `./nope` and not `./internal/x`; `-C sub ./pkg` with `sub` and `pkg` present in the cwd but no `sub/pkg` refuses naming `./pkg`; `./nope/...` refuses; `./nope ./gone` refuses naming both.
  Each row keeps the existing exit 1, `ok: false` and way-forward assertions.
- `TestGateTest_Scenario`'s existing subtests name `./pkg`, `./held`, `./nested` and `./enclosed` that exist nowhere, which the refusal would now stop.
  The test creates those directories in the prime once at its top, and the two subtests that run from a `t.TempDir()` cwd (the unslotted run outside every hub and the tagged run in a root without `cmd/lyx`) create `pkg` in that cwd first.
  A new subtest "a missing package inside the hub refuses before any slot or build" runs `test --tags integration ./nope` from the prime, with `cmd/lyx` present there as the tagged-build subtest makes it, and asserts exit 1, the JSON error naming `./nope`, no stand-in record (so neither the prebuild nor go test ran) and `requireFree`, so the check is pinned ahead of the slot and the prebuild.
  A new subtest "import paths, patterns and .go files reach go test" runs `test ./... ./pkg/... example.com/m/a ./absent.go` from the prime and asserts the stand-in's recorded args carry those four arguments unchanged.

Docs in the same commit: `internal/gatecli/doc.go`'s Verb section states the check, its `_test` hint and its bound, and `contracts/specs/refusal-spec.md`'s gate table gains the row `package directory missing`, trigger "a relative package argument, its trailing `/...` removed, names no existing directory under the run directory", class correctness halt like the no-package row, way forward "re-run the same command with corrected package arguments, the external-test directory for a `_test` argument".

**ImpactSummary:** One new refusal inside `runTest` ahead of hub resolution; the integration scenario's placeholder package arguments gain real directories, and every gate run Webster derives already names existing directories after card 1.

**Commit:** `3: gate-package-refusal`
