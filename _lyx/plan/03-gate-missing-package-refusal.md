# Card 3 — gate-missing-package-refusal

**Create:**
- `plan:internal/gatecli#missingPackageArgs` -> `func missingPackageArgs(dir string, packages []string) []string`
- `plan:internal/gatecli#refuseMissingPackages` -> `func refuseMissingPackages(out io.Writer, dir string, missing []string) int`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#newCommand`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Intent:**
D4: `lyx gate test` refuses a relative package argument that names no directory, before any slot and before the prebuild.

Both new functions live in `internal/gatecli/cli.go` beside `refusePackages`.
`missingPackageArgs` returns, in order, each package argument spelled relative (`.`, `..`, `./…`, `../…`) and not ending in `.go` whose path, with any trailing `/...` removed, joined onto `dir` is not an existing directory.
Import paths, absolute paths and `.go` file arguments are never returned, so they pass to go test unchanged.
`refuseMissingPackages` writes the JSON error envelope through `output.Err` and returns its exit 1.
The message opens `gate test: `, holds one clause per missing argument, `<arg> names no directory under <dir>`, joined by `; `, and ends `; way forward: re-run the same command with corrected package arguments`.
For an argument ending in `_test` (after the `/...` trim) whose trimmed directory exists, its clause adds `: it is an external test package, whose tests live in <corrected>`, `<corrected>` being the argument with that suffix removed.
This trims a CLI path argument, never a glyph, so `PATTERN-glyph-conversion-chokepoint` does not bind it.

`runTest` calls both right after `absoluteDir` resolves the run's directory (the `-C` value against the seam cwd, or the seam cwd itself, never `os.Getwd`) and before `catchTerminatingSignals` and `resolveHub`, so a refused request takes no slot, inherited or acquired, runs no prebuild and runs no package.
Every upstream check passes for an input with at least one package and an existing `-C` directory, so the refusal is reached by any such request naming a missing relative directory.

`TestGateTest_RefusesBeforeResolvingTheHub` gains a `dirs` field, the directories each row creates under its own `t.TempDir()` cwd, and a `wantAbsent` field, a fragment the error must not hold; the existing rows keep empty values.
New rows, each expecting exit 1, ok false, the fragment and a way forward:
- `./internal/x_test` with `internal/x` present: the error holds `whose tests live in ./internal/x`;
- `./nope ./internal/x` with `internal/x` present: the error holds `./nope names no directory`, and `wantAbsent` is `./internal/x names`;
- `-C sub ./nope` with `sub` and `nope` present in the cwd: the error holds `./nope names no directory`, since the argument resolves against `sub`;
- `./missing/...`: the error holds `./missing/... names no directory`, since a clause names the argument as written.

`TestGateTest_Scenario` creates the directory `pkg` in the prime once, at its start, since every row passes `./pkg` and would otherwise now be refused.
Its two subtests that run from a `t.TempDir()` cwd outside every hub with `./pkg`, `a target outside every hub runs unslotted and logs it` and `a tagged run in a root without cmd/lyx builds nothing and strips an inherited prebuilt variable`, create `pkg` in that cwd first.
Its `refusals` table gains an `args` field, defaulting to `test ./pkg` when empty, and the row `a missing package directory refuses before any slot or prebuild`: from the prime, `test --tags integration ./nope` exits 1 with an error holding `./nope names no directory`, the stand-in records nothing (neither the prebuild nor go test ran), and `requireFree` sees no holder or wait record, so a check placed after `acquireSlot` or `prebuildLyx` fails it.
It gains the subtest `a pattern, an import path and a .go file argument reach go test`: from the prime, `test ./pkg/... example.com/m/x ./x.go` exits 0 and the stand-in records exactly `test -C <prime> -p 3 ./pkg/... example.com/m/x ./x.go`.

`contracts/specs/refusal-spec.md`'s gate table gains the row `package argument names no directory` after `-C directory missing`: trigger, a relative package argument not ending in `.go`, with any trailing `/...` removed, that names no existing directory under the run's directory, checked before any slot or prebuild; class correctness halt, like the no-package row; way forward, re-run the same command with corrected package arguments, the error naming an external test package's directory for a `<dir>_test` argument.

Docs: `internal/gatecli/doc.go` states the refusal, its place before hub resolution, and what passes unchecked.
`newCommand`'s `Long` text, which says the output and exit code are go test's own, gains one line: a relative package argument naming no directory is refused before any slot.
`docs/overview.md`'s gate row, which says the verb exits with go's code, gains the same clause.

The bound: a request whose one wrong argument used to run the other packages and fail on it now runs none.

**ImpactSummary:** every `lyx gate test` naming a missing relative package directory refuses before any slot or prebuild instead of running go test; the integration scenario now creates the `pkg` directory its rows name.

**Commit:** 3: gate: refuse a relative package argument naming no directory
