# Card 2 — verify-package-missing

**Create:**
- `internal/planparser/verifypackage.go`
- `plan:internal/planparser#checkVerifyPackageMissing` -> `func checkVerifyPackageMissing(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser#placedPackageArg` -> `type placedPackageArg struct`
- `plan:internal/planparser#placedPackageArgs` -> `func placedPackageArgs(chain string) []placedPackageArg`
- `plan:internal/planparser#gateTestDirAndPackages` -> `func gateTestDirAndPackages(runDir string, fields []string) (commandDir string, packageArgs []string, ok bool)`
- `plan:internal/planparser_test#TestValidate_VerifyPackageMissing` -> `func TestValidate_VerifyPackageMissing(t *testing.T)`

**Edit:**
- `internal/planparser#goCommandDirAndPackages` -> `func goCommandDirAndPackages(runDir string, fields []string) (commandDir, subcommand string, packageArgs []string, ok bool)`
- `internal/planparser#crossModuleCommands`
- `internal/planparser/validate.go`
- `internal/planparser/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D3: a new blocking plan check, `verify-package-missing`, reports an author-written verify argument that names no directory.

`internal/planparser/verifypackage.go` holds the check and its chain walk.
`placedPackageArgs` walks one chain with `splitChain`, skips leading environment assignments with `isEnvAssignment`, and tracks the run directory exactly as `crossModuleCommands` does: it starts at `.`, follows a literal `cd <dir>` through `resolveRunDir`, and ends the walk (returning what it found so far) at a `cd` with other than one argument, a `cd` in a pipeline stage or after `||`, or a `cd` or `-C` it cannot place.
It returns one `placedPackageArg` per package argument, with fields `Arg` (the argument as written) and `Dir` (the worktree-relative directory it names: the argument joined onto the command's directory with `path.Join`, after any trailing `/...` is removed), from two command shapes:
- `go … build|test|vet …`, read through `goCommandDirAndPackages`, whose new `subcommand` result is the field after the leading flags (empty when there is none); any other subcommand contributes nothing.
- `lyx gate test …`, read through `gateTestDirAndPackages`, which takes the fields after `test`: `-C <dir>`, `-C=<dir>`, `--dir <dir>` and `--dir=<dir>` set the command directory through `resolveRunDir` (ok false when it cannot be placed), `--tags <v>` and `--tags=<v>` are skipped, and the walk of the command stops at `--`.
For both shapes a package argument is a field spelled `.`, `..`, or opening with `./` or `../`, and an argument ending in `.go` is dropped.
So an import path, an absolute path, an argument behind a variable or inside `bash -c`, and every argument after an unplaceable `cd` pass, which is D3's bound; `./...` joins to the command's own directory, which exists, so it passes too.

`goCommandDirAndPackages` gains the `subcommand` result; its one caller, `crossModuleCommands`, ignores it, so verify-nested-module is unchanged.

`checkVerifyPackageMissing` reads each card's `Verify` value and the overview's `plan.Verify`.
The created directories for a card numbered N are the `Dir` of every entry `CardTargetDirs` yields with `DeleteOnly` false for each card numbered at or below N; for the overview chain they span the whole plan.
That makes a path or glyph target of any non-Delete group a created directory, member glyphs and Custom groups included, and a Rename pair's New side, since `CardTargetDirs` already reads Rename groups by their New side.
A card's Verify is skipped when that card or any earlier card carries a `Move` group, and the overview chain is skipped when any card does, since a Move's destination lives in Intent prose.
An argument is a finding when its `Dir` neither is a directory on disk under `worktreeRoot` (`os.Stat` and `IsDir`) nor equals, or is an ancestor of, a created directory (`created == dir` or `strings.HasPrefix(created, dir+"/")`, with `.` an ancestor of every directory).
A directory an earlier card deletes still exists on disk at plan time and is not reported; the card's own verify run catches it, as D3's bound says.

Each finding has Check `verify-package-missing`, Card the card's `cardID` (empty for the overview), Ref the argument, and a Detail naming the card or "the overview's `## verify:` section", the command's argument, and that it names no directory on disk or created at or before that card, ending with the way forward "correct the Verify line".
When the argument's last segment, after `/...` is removed, ends in `_test` and the directory with that suffix removed exists on disk or is created, the Detail names it as an external test package whose tests live in that directory and names the corrected argument in backticks, for example `` `./internal/x` ``.

In `internal/planparser/validate.go`, listed as a file because the card changes both its package comment and its `validate` function: `validate` appends `checkVerifyPackageMissing(plan, worktreeRoot)` right after `checkVerifyModuleWide`, so both `Validate` and `ValidateFormat` run it; the file's package comment lists the new ID after `verify-module-wide (checkVerifyModuleWide, verifymodule.go)` as `verify-package-missing (checkVerifyPackageMissing, verifypackage.go)`.
Dispatch already drops a finding against a completed card, so a card whose verify ran is never re-judged.

`TestValidate_VerifyPackageMissing` is a new top-level table test in `internal/planparser/validate_test.go`, beside `TestValidate_VerifyModuleWide`, filtering `ValidateFormat` findings by Check and comparing the Card list, plus a Detail assertion where the row names one.
It runs over a `t.TempDir()` root on which `materializeFiles` writes `internal/x/x.go` and `sub/pkg/p.go`, and over cards built from `validCard` and `cardOfType`.
Reported rows:
- a card Verify `lyx gate test ./internal/x_test`, whose Detail holds the phrase `external test package` and the corrected argument as a bounded token, `` `./internal/x` `` with its backticks, which the argument `./internal/x_test` alone cannot produce;
- card 1 creating `internal/y/y.go`, then card 2's Verify `go test ./internal/y_test`, whose Detail holds `external test package` and `` `./internal/y` ``, the hint for a created directory;
- card 1 whose only target is the `Delete` of `internal/gone/g.go`, absent on disk, then card 2's Verify `go test ./internal/gone`, reported because a `DeleteOnly` directory is no created directory;
- the overview chain `go test ./nope`, reported with an empty Card;
- `lyx gate test -C sub ./nope` and `go -C sub test ./nope`, each reported, since `sub/nope` does not exist while `nope` is judged under `sub`;
- card 1's Verify `go test ./internal/new` where only card 2 creates `internal/new/x.go`.
Silent rows:
- `go test ./internal/x` and `lyx gate test --tags integration ./internal/x`, an existing directory;
- a card whose own Edit target is `internal/new/y.go` with Verify `go test ./internal/new`;
- a Create card targeting `internal/new/x.go#Foo` followed by a card whose Verify is `go test ./internal/new`;
- a Custom card targeting `internal/new/x.go` followed by a Verify on `./internal/new`;
- a Rename card whose pair is `internal/x/x.go#Old` -> `plan:internal/new#New`, followed by a Verify on `./internal/new`;
- `go test ./...`, `go vet example.com/mod/pkg`, `go run ./nope`, `go test ./nope/a_test.go`, and `cd /abs && go test ./nope`;
- a Move card followed by a card whose Verify is `go test ./nope`, and the overview chain `go test ./nope` in a plan holding that Move card.
The created-directory ancestor row: card 2 creating `internal/new/deep/x.go`, then card 3's Verify `go test ./internal/new/...`, is silent.

`cmd/lyx/testdata/test-budget.yaml`'s `internal/planparser` row rises from 92 to 93 for the new top-level test.

Docs: `internal/planparser/doc.go`'s "Verify chains" section names `verify-package-missing` as a third reader of the shared splitter and states which commands and arguments it judges and its bound.
`contracts/specs/loom-plan-spec.md` gains a row `42. \`verify-package-missing\`` after row 41: the commands and argument spellings it reads, the `cd` and `-C` tracking it shares with `verify-nested-module`, the existence rule and the created directories, the `_test` replacement hint, the way forward "correct the Verify line", that it runs under any `language:` in `Validate` and `ValidateFormat` after `verify-module-wide` and is blocking, and its bound: static spellings only, nothing checked on or after a Move card, and a directory an earlier card deletes is not reported.

**ImpactSummary:** One new blocking check in `validate`; `goCommandDirAndPackages`'s only caller `crossModuleCommands` ignores the added result, so verify-nested-module is unchanged.

**Commit:** `2: Plan check verify-package-missing for verify arguments that name no directory`
