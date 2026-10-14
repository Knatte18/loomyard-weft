# Card 3 — gate-test-missing-dir

**Create:**
- `plan:internal/gatecli#refuseMissingPackageDirs` -> `func refuseMissingPackageDirs(out io.Writer, dir string, packages []string) (code int, refused bool)`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:** D4: `lyx gate test` refuses a relative package argument that names no directory, before any slot or prebuild.

`refuseMissingPackageDirs`, in `internal/gatecli/cli.go` beside `refusePackages`, checks each of `packages` spelled relative (`.`, `..`, or opening with `./` or `../`) and not ending in `.go`: with any trailing `/...` removed, it is joined onto `dir` with `filepath.Join` and `filepath.FromSlash`, and is missing when `os.Stat` fails or reports no directory.
With none missing it returns `0, false`.
Otherwise it writes one JSON error through `output.Err` and returns `1, true`.
The message is `gate test: <args> name no directory under <dir>`, listing every missing argument in order and no other;
for each missing argument whose last segment ends in `_test` and whose directory with that suffix removed exists, it adds that it is an external test package whose tests live in that directory and names the corrected argument in double quotes, for example `"./internal/x"`;
it ends `; way forward: re-run the same command with each argument naming an existing package directory`, naming the corrected arguments where a hint gave one.

`runTest` calls it right after `absoluteDir` resolves `dir` (the `-C` value against the seam cwd from `lyxcwd.CwdFrom`, or the seam cwd itself, never `os.Getwd`), and returns its code when it refuses; that is before `catchTerminatingSignals`, `resolveHub`, any slot path (inherited, slotted or outside every hub) and the prebuild.
Bound: import paths, absolute paths, `.go` file arguments and patterns pass to go test unchanged, and `./…/...` is judged by the directory before `/...`; a refused request runs no package.
The way forward needs no denied command.

Tests:
- `TestGateTest_RefusesBeforeResolvingTheHub` gains a `dirs []string` field, created under the row's cwd before the run, and a `notWant string` field the error must not hold.
  New rows: `./internal/x_test` with `internal/x` created, wanting `external test package` and `"./internal/x"` with its quotes in the error, which the listed argument `./internal/x_test` alone cannot produce; `./nope ./internal/x` with `internal/x` created, wanting `./nope` and, as `notWant`, `./internal/x`; `-C sub ./nope` with `sub` created, wanting `./nope` and, as `notWant`, `does not exist`, so the row reaches this refusal rather than the `-C directory missing` one.
  Every row already asserts exit 1, `ok: false` and a way forward.
  These rows run outside every hub and untagged, so they cannot show the refusal precedes the slot and the prebuild; the scenario row below does.
- `TestGateTest_Scenario` is forced: its rows run `./pkg`, `./held`, `./nested` and `./enclosed` in the prime, and `./pkg` in a fresh `t.TempDir()` cwd, none of which exist, so D4 would refuse them.
  At its start it creates `pkg`, `held` and `enclosed` under the prime with `os.MkdirAll` (`nested` is already created before the rows that use it as `nestedModule`, and the inherited-slot row's `./nested` runs before that, so `nested` is created up front too), and the two rows whose cwd is a fresh `t.TempDir()` with `./pkg` (the outside-every-hub row and the tagged root-without-cmd/lyx row) first create `pkg` in that directory.
  The existing row "a target outside every hub runs unslotted and logs it" also creates `internal/x` in its cwd, extends its args to `test ./pkg ./internal/... example.com/imp ./internal/x/x_test.go`, and asserts the stand-in args end in exactly those four arguments, so a pattern, an import path and a `.go` file argument reach go test.
  A new subtest "a package argument that names no directory refuses before the slot and the prebuild" runs in the prime with `test --tags integration ./nope` and asserts exit 1, an error naming `./nope`, no stand-in record at all (`g.record` absent, so neither the prebuild's `go build` nor `go test` ran), and `requireFree`, so no slot was taken or left; the existing "-C names a worktree from a cwd outside the hub" row now pins that `-C` arguments resolve against the `-C` directory, since `./pkg` exists under the prime and not under its cwd.

Docs: `internal/gatecli/doc.go`'s Verb section states the refusal, its message, its hint and its bound.
`contracts/specs/refusal-spec.md`'s `## gate` table gains, after the `-C directory missing` row, a row `package directory missing`: `lyx gate test` names a relative package argument, not a `.go` file, whose directory (trailing `/...` removed) does not exist under the run directory, checked before any slot or prebuild; correctness halt, like the no-package row; way forward: re-run the same command with each argument naming an existing package directory, the external test package's own directory for a `_test` argument.

**ImpactSummary:** `runTest` gains one refusal before hub resolution; every caller passing an existing directory is unchanged, and the scenario fixture is the one test forced to create its directories.

**Commit:** `3: lyx gate test refuses a package argument that names no directory (#571, #577, #580, #587)`
