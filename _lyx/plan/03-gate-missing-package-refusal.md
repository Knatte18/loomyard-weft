# Card 3 — lyx gate test refuses a package argument that names no directory

**Create:**
- `plan:internal/gatecli#missingPackageArgs` -> `func missingPackageArgs(dir string, packages []string) []string`
- `plan:internal/gatecli#refuseMissingPackages` -> `func refuseMissingPackages(out io.Writer, dir string, missing []string) int`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`
- `contracts/specs/refusal-spec.md`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`

**Intent:**
D4: the issues ask for the missing directory to be named, and refusing before the slot saves the wait and the prebuild of a run that cannot pass.

`missingPackageArgs`, in `cli.go`, returns, in order, each of `packages` spelled relative (`.`, `..`, `./…`, `../…`) and not ending in `.go` whose path, any trailing `/...` removed and joined onto `dir`, is not an existing directory.
An import path, an absolute path, a pattern whose base exists and a `.go` file argument are never returned, so they reach go test unchanged.

`refuseMissingPackages`, in `cli.go`, writes the refusal through `output.Err` and returns its exit code, 1.
Its message is `gate test: package argument <args> names no directory under <dir>`, the arguments joined by `, `; for each argument ending in `_test` whose `_test`-trimmed path is an existing directory under `dir`, it adds `; <arg> is an external test package, whose tests live in <trimmed>; name <trimmed>`; it ends with `; way forward: re-run the same command with each package argument naming an existing directory`.
The trim is a plain path-suffix trim on a command-line argument, not a glyph conversion, so `PATTERN-glyph-conversion-chokepoint` does not apply.

`runTest` calls `missingPackageArgs(dir, request.packages)` right after `absoluteDir` resolves `dir` (the `-C` value against the seam cwd, or the seam cwd), and before `catchTerminatingSignals`, `resolveHub`, any slot path, inherited, slotted or outside every hub, and the prebuild; a non-empty result returns `refuseMissingPackages`.
The input that reaches it after every upstream check passes: at least one package, an existing `-C` directory, and a relative argument such as `./internal/x_test` naming no directory under it.
A refused request runs no package, even one that names an existing directory beside the missing one.

Tests:
- `TestGateTest_RefusesBeforeResolvingTheHub` runs each row in a fresh cwd holding the directory `internal/x`, and gains a `wantAbsent` field the error must not contain.
  New rows: `./internal/x_test` refuses with an error holding `./internal/x_test is an external test package, whose tests live in ./internal/x; name ./internal/x`; `./nope ./internal/x ./gone` refuses with an error holding `./nope, ./gone names no directory`, every missing argument named, with `wantAbsent` `./internal/x`; `-C internal ./internal/x` refuses with an error holding `./internal/x names no directory`, since it resolves to the absent `internal/internal/x`.
  The existing rows keep their expectations, each refused before the new check or by the `-C` check.
- `TestGateTest_Scenario` creates the directories `pkg`, `held`, `nested` and `enclosed` under the prime worktree before its first subtest, since its stand-in runs name them and the gate now refuses an absent one; its two subtests that run `./pkg` from a fresh `t.TempDir()` cwd create `pkg` in that cwd first.
  A new subtest runs `test --tags integration ./absent ./pkg` in the prime: exit 1, an error naming `./absent` and not `./pkg`, no stand-in record (no lyx prebuild and no go test ran), and the slot and wait directory left free.
  A new subtest runs `test ./pkg/... example.com/m/x ./missing_test.go` in the prime: exit 0, and the stand-in's args are `test -C <prime> -p 3 ./pkg/... example.com/m/x ./missing_test.go`.
  It exists for what no stand-in subtest reaches yet: an import path, a `.go` argument naming no file and a pattern over an existing base, each passed through the new check unchanged; `./...` alone is already reached by the nested-module subtest.
  The existing `-C names a worktree from a cwd outside the hub` subtest then shows a `-C` argument resolving against that directory, since its cwd holds no `pkg`.

`contracts/specs/refusal-spec.md`'s `lyx gate test` table gains a row `package argument names no directory`, a correctness halt like the no-package row, whose way forward is to re-run the same command with each package argument naming an existing directory, an external test package named by its package directory.
`tools/sandbox/SANDBOX-CORE-SUITE.md`'s S10 gains a Watch line: does `lyx gate test ./no-such-dir` print one `{"ok":false}` envelope whose `error` names `./no-such-dir` and a way forward, and exit 1 without running go test?
`internal/gatecli/doc.go` states the refusal, where it sits in the sequence and its bound: import paths, absolute paths, patterns over an existing directory and `.go` file arguments pass to go test unchanged.

**ImpactSummary:** a hand-written gate line with one relative argument naming no directory turns from a partial run into a refusal before the slot; every other request runs as before.

**Commit:** 3: refuse a gate test package argument that names no directory
