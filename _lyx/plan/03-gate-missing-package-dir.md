# Card 3 — gate-missing-package-dir

**Create:**
- `plan:internal/gatecli#missingPackageDirs` -> `func missingPackageDirs(dir string, packages []string) []string`
- `plan:internal/gatecli#refuseMissingPackageDirs` -> `func refuseMissingPackageDirs(out io.Writer, dir string, missing []string) int`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:** `lyx gate test` refuses a relative package argument that names no directory (D4), so a hand-written gate line such as `lyx gate test ./internal/x_test` fails fast, naming the directory to use, instead of waiting for a slot and a prebuild of a run that cannot pass.

- `missingPackageDirs` lives in `cli.go` beside `absoluteDir` and returns, in argument order, every package argument spelled relative (`.`, `..`, `./…`, `../…`) and not ending in `.go` whose path, joined onto `dir` with a trailing `/...` removed, is not an existing directory.
  Any other argument (an import path, an absolute path, a pattern that is not relative such as `all`, a `.go` file) is never checked.
- `refuseMissingPackageDirs` writes one JSON error through `output.Err` naming every missing argument and returns 1.
  For an argument ending in `_test`, once `/...` is removed, whose trimmed path is an existing directory under `dir`, the message says it is an external test package whose tests live in that directory and names the corrected argument (`./internal/x` for `./internal/x_test`).
  The message ends "way forward: re-run the same command with corrected package arguments".
- `runTest` calls them right after `absoluteDir` resolves the directory (the `-C` value against the seam cwd, or the seam cwd itself; never `os.Getwd`) and before `catchTerminatingSignals`, hub resolution, the inherited, slotted and unslotted paths and the prebuild, so a refused request takes no slot, writes no wait record, builds nothing and runs no package.

Bound: only a relative spelling is checked, against the directory the run uses; import paths, absolute paths, non-relative patterns and `.go` file arguments pass to go test unchanged.
A refused request runs nothing, where today a wrong argument among right ones gave a partial run that go test failed.

Tests:

- `TestGateTest_RefusesBeforeResolvingTheHub` gains a `dirs` field, directories the row creates under its cwd before the run, and a `notWant` field, a substring the error must not hold.
  New rows: `test ./internal/x_test` with `internal/x` present refuses naming `./internal/x` and "external test package";
  `test ./nope ./gone ./internal/x` with `internal/x` present refuses naming both `./nope` and `./gone` (the row's `wantError` holds `./nope` and a second substring field `wantAlso` holds `./gone`), with `notWant` `"./internal/x"`;
  `test -C sub ./internal/x_test` with `sub/internal/x` present and no `internal/x` under the cwd refuses naming `./internal/x`, resolved against `sub`.
  The existing assertions (exit 1, `ok` false, the substring and a way forward) apply to every row.
- `TestGateTest_Scenario` (integration tier) keeps reaching its stand-in go binary: right after `prime` is known it creates the `pkg`, `held`, `nested` and `enclosed` directories under `prime`, and each subtest whose cwd is a fresh `t.TempDir()` and whose arguments name `./pkg` (the unslotted outside-every-hub run and the tagged run in a root without `cmd/lyx`) creates `pkg` in that cwd first.
  The outside-every-hub subtest also creates `internal` in its cwd and runs `test ./internal/... example.com/x ./a.go ./pkg`, asserting the stand-in's recorded args end with exactly those four arguments, so a relative pattern over an existing directory, an import path and a `.go` file argument reach go test unchanged.
  The `refusals` table gains an `args` field defaulting to `test ./pkg` and a row "a package argument naming no directory refuses before the slot and the prebuild" with args `test --tags integration ./absent_test` and `wantError` `./absent_test`.
  Its setup holds the hub's one slot through `pool.Acquire` and its restore releases it, so a refusal placed after the slot acquire would wait out `cli_wait_sec` and exit `SlotBusyExit` instead of 1;
  the table's existing assertions (exit 1, no stand-in record, so neither a prebuild nor a go test ran, and the slot free with no wait record after the restore) then pin the refusal ahead of both the slot and the prebuild.

Docs in this commit:

- `contracts/specs/refusal-spec.md`'s `## gate` table gains a row after `-C directory missing`: "package directory missing", triggered when a relative package argument not ending in `.go` names no existing directory under the run's directory, with the external-test hint, class correctness halt like `no package argument`, way forward "re-run the same command with corrected package arguments".
- `internal/gatecli/doc.go` states the refusal, where it sits in `runTest`'s order, and the bound.

**ImpactSummary:** Adds one refusal ahead of every slot path in `runTest`; a gate line whose relative arguments all name directories, or that names only import paths, patterns over existing directories or `.go` files, runs exactly as before.

**Commit:** `3: lyx gate test refuses a package argument that names no directory`
