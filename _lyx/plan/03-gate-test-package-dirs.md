# Card 3 — gate-test-package-dirs

**Create:**
- `internal/gatecli#missingPackageDirs`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:** D4: `lyx gate test` refuses a relative package argument that names no directory, before any slot path and before the prebuild.

`missingPackageDirs` goes in `internal/gatecli/cli.go`, beside `absoluteDir`.
It reads each package argument spelled relative (`.`, `..`, `./…`, `../…`) and not ending in `.go`, removes a trailing `/...`, joins the rest onto `dir` and stats it.
It returns "" when every such argument is an existing directory, and otherwise the refusal text naming every argument that is not, in argument order: `gate test: package argument ./nope names no directory under <dir>`, joined with "; " for several.
For an argument whose trimmed spelling ends in `_test` while the `_test`-less directory exists, its clause reads instead `package argument ./internal/x_test is an external test package whose tests live in ./internal/x; pass ./internal/x`.
The text ends with `; way forward: re-run the same command with each named package argument corrected`.
Import paths, absolute paths, module-wide patterns whose stem exists and `.go` file arguments are not read, and so reach go test unchanged.

`runTest` calls it right after `absoluteDir` resolves `dir` (the `-C` value against the seam cwd, or the seam cwd itself, never `os.Getwd`, per `PATTERN-cwd-resolution`), and returns `output.Err(out, text)`, exit 1, on a non-empty result.
That is before `catchTerminatingSignals`, `resolveHub`, any slot acquisition, inherited, slotted or outside every hub, and the `--tags` prebuild, so a refused request runs no package and waits for no slot.

Bound: the refusal adds a guard and lets nothing new through; an argument the check does not read passes to go test exactly as today.
A hand-written gate line with one wrong argument now refuses instead of running its other packages.

Tests:
- `TestGateTest_RefusesBeforeResolvingTheHub` gains a `dirs []string` field, created under the row's own cwd before the run, and a `wantAbsent string` the error must not contain; the cwd stays a `t.TempDir()` outside every hub, so a refusal there is reached before hub resolution.
  New rows: "external test package names its package directory" (`dirs` `internal/x`, args `test ./internal/x_test`, error holding `./internal/x;`, the corrected argument); "only the missing argument is named" (`dirs` `internal/x`, args `test ./nope ./internal/x`, error holding `./nope`, `wantAbsent` `package argument ./internal/x `); and "-C arguments resolve against that directory" (`dirs` `sub/x`, args `test -C sub ./x_test`, error holding `pass ./x`).
  Each also asserts the existing exit 1, `ok` false and way forward.
- `TestGateTest_Scenario` names `./pkg`, `./held`, `./nested` and `./enclosed`, directories neither the hubforge prime nor a bare `t.TempDir()` holds, so it creates `pkg`, `held`, `nested` and `enclosed` under `prime` at setup, and gives each row whose cwd is a `t.TempDir()` and whose argument is `./pkg` a cwd holding `pkg`: "a target outside every hub runs unslotted and logs it" and "a tagged run in a root without cmd/lyx builds nothing and strips an inherited prebuilt variable".
  Its `refusals` table gains an `args` field, defaulting to `test ./pkg` for the existing rows, and the row "a missing package directory refuses before the slot and the prebuild", with a no-op setup and args `test --tags integration ./nope` in `prime`.
  The table's own assertions then pin the in-hub ordering: exit 1 with an error holding `./nope`, no stand-in record, so neither the prebuild nor go test ran, and `requireFree`, so no slot is held.
  The "a target outside every hub" row then runs `test ./pkg/... example.com/x ./absent/a_test.go` and asserts the stand-in received those three arguments after the `-p` cap, pinning that a pattern, an import path and a `.go` file argument reach go test.

Docs: `contracts/specs/refusal-spec.md`'s gate table gains the row `package directory missing`: a relative package argument, a trailing `/...` removed and not a `.go` file, names no directory under the run directory, the error naming each such argument and the external-test replacement; correctness halt like the no-package row; way forward "re-run the same command with each named package argument corrected".
`internal/gatecli/doc.go`'s Verb section states the check, its place before hub resolution and the arguments it leaves alone.

**ImpactSummary:** one refusal added inside `runTest` before hub resolution; every caller whose relative package arguments name directories, which includes webster's derived gates once card 1 lands, runs as before, and the integration scenario gains the directories its arguments name.

**Verify:** lyx gate test --tags integration ./internal/gatecli

**Commit:** 3: lyx gate test refuses a relative package argument that names no directory
