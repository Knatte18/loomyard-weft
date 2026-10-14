# Card 3 — gate-package-dir-refusal

**Create:**
- `plan:internal/gatecli#packageDirRefusal` -> `func packageDirRefusal(dir string, packages []string) string`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/gatecli#absoluteDir`

**Intent:** D4: `lyx gate test` refuses a relative package argument that names no directory, naming the missing directory as #571, #577, #580 and #587 ask, before it waits for a slot or prebuilds.

`packageDirRefusal` checks each package argument spelled relative (`.`, `..`, `./…`, `../…`) and not ending in `.go`, with any trailing `/...` removed, joined onto `dir`.
An argument whose joined path is not an existing directory is missing.
It returns "" when none is missing, and otherwise the refusal text: `gate test: ` plus every missing argument as typed, each with the directory it was looked for in, then `; way forward: re-run the same command with corrected package arguments`.
For a missing argument ending in `_test` whose trimmed directory exists, the text says it is an external test package whose tests live in that directory and names the corrected argument, for example `./internal/x`.

`runTest` calls it right after `absoluteDir` resolves the run directory, the `-C` value or the seam cwd from `lyxcwd.CwdFrom`, never `os.Getwd`, and before `catchTerminatingSignals`, `resolveHub`, any slot path (inherited, slotted or outside every hub) and `prebuildLyx`; a non-empty result is `output.Err`, exit 1, so a refused request runs no package.
The input that reaches it after every upstream check passes is at least one package argument and an existing `-C` directory, with a relative argument naming no directory under it.

Bound: import paths, absolute paths, patterns that resolve to an existing directory and `.go` file arguments pass to go test unchanged; the check reads only the arguments before `--`.

Tests:
- `TestGateTest_RefusesBeforeResolvingTheHub` gains a per-row `dirs` list created under the row's cwd and a `wantAbsent` string the error must not hold.
  New rows: `./internal/x_test` with `internal/x` present refuses, holding `./internal/x` and `external test package`; `./nope ./internal/x` with `internal/x` present refuses, holding `./nope` and not holding `./internal/x`; `-C sub ./x` with `sub` and `x` present under the cwd refuses holding `./x`, which proves the argument resolves against the `-C` directory.
  Each refusal row already asserts exit 1, `ok` false and a way forward.
- `TestGateTest_Scenario` creates `pkg`, `held`, `nested` and `enclosed` under the prime before its subtests, since its rows name those directories and the refusal would now stop them; the two subtests that run from a fresh `t.TempDir()` cwd with `./pkg` create `pkg` there.
  The scenario's `refusals` table gains a row whose arguments are `--tags integration ./nope` from the prime, with `cmd/lyx` present there so the prebuild would otherwise run, and whose setup holds the pool's one slot through `pool.Acquire` and releases it in its restore, holding `./nope` in its error.
  That table's loop already asserts exit 1 and no stand-in record: exit 1 rather than `SlotBusyExit` pins that the refusal precedes the slot wait, and no record pins that neither the `lyx` build nor go test ran.
  The table's per-row `args` become a field, defaulting to `./pkg`, so the new row can carry its own.
  A new subtest runs from the prime with `internal` created and the arguments `./internal/... example.com/m/pkg ./main.go`, and asserts exit 0 and that the stand-in's go args carry all three unchanged after `-p 3`, so a pattern, an import path and a `.go` file reach go test.

`internal/gatecli/doc.go`'s Verb section states the check, its place before the slot and the prebuild, and its bound.
`contracts/specs/refusal-spec.md`'s gate table gains a row after `-C directory missing`: `package directory missing`, triggered by a relative package argument that names no existing directory under the run directory, with the external-test hint, class correctness halt like the no-package row, way forward `re-run the same command with corrected package arguments`.

**ImpactSummary:** One refusal inside `runTest` ahead of hub resolution; its only callers are the cobra verb and the test seams, and the integration scenario's fixtures gain the directories they name.

**Commit:** `3: lyx gate test refuses a package argument that names no directory`
