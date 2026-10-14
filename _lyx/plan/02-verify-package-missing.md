# Card 2 — verify-package-missing

**Create:**
- `plan:internal/planparser#walkChain` -> `func walkChain(chain string, visit func(runDir string, fields []string) bool)`
- `plan:internal/planparser#packageArgument` -> `type packageArgument struct`
- `plan:internal/planparser#relativePackageArgs` -> `func relativePackageArgs(chain string) []packageArgument`
- `plan:internal/planparser#gateTestDirAndPackages` -> `func gateTestDirAndPackages(runDir string, fields []string) (commandDir string, packageArgs []string, ok bool)`
- `plan:internal/planparser#checkVerifyPackageMissing` -> `func checkVerifyPackageMissing(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser_test#TestValidate_VerifyPackageMissing` -> `func TestValidate_VerifyPackageMissing(t *testing.T)`

**Edit:**
- `internal/planparser#crossModuleCommands`
- `internal/planparser#goCommandDirAndPackages` -> `func goCommandDirAndPackages(runDir string, fields []string) (commandDir, subcommand string, packageArgs []string, ok bool)`
- `internal/planparser#validate`
- `internal/planparser/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `plan:internal/planparser#externalTestPackageDir`
- `internal/planparser#CardTargetDirs`
- `internal/planparser#splitChain`
- `internal/planparser#isEnvAssignment`
- `internal/planparser#resolveRunDir`
- `internal/planparser_test#validCard`
- `internal/planparser_test#cardOfType`
- `internal/planparser_test#materializeFiles`

**Intent:**
D3: the new blocking plan check `verify-package-missing` refuses a relative package argument in an author-written verify chain that names no directory.

Everything new lives in `internal/planparser/verifymodule.go`, whose file comment then names all three verify-chain checks.

`walkChain` is the chain walk `crossModuleCommands` does today, lifted out: it splits with `splitChain`, drops leading `isEnvAssignment` fields, follows a literal `cd <dir>` through `resolveRunDir` after a sequencing separator, and ends the walk at a `cd` it cannot place or one in a pipeline stage or after `||`.
For every other non-empty segment it calls `visit` with the run directory and the fields, and stops when `visit` returns false.
`crossModuleCommands` keeps its behaviour and becomes a `walkChain` visitor, which `TestValidate_VerifyNestedModule` pins unchanged.

`goCommandDirAndPackages` also returns the subcommand it reads (empty when the fields end before one); `crossModuleCommands`, its only caller, ignores it.

`gateTestDirAndPackages` reads the fields after `lyx gate test`: `-C <dir>`, `-C=<dir>`, `--dir <dir>` and `--dir=<dir>` set the command directory through `resolveRunDir`, `--tags <v>` and `--tags=<v>` are skipped with their value, `--` ends the packages, and a relative field (`.`, `..`, `./…`, `../…`) is a package argument.
ok is false for a directory it cannot place or any other flag.

`packageArgument` holds a package argument as written (`Arg`) and the worktree-relative directory it names (`Dir`): the command directory joined with the argument, with a trailing `/...` removed.
`relativePackageArgs` walks a chain with `walkChain` and yields one `packageArgument` per relative package argument of a `lyx gate test` and of a `go build`, `go test` or `go vet` (any other `go` subcommand yields none), skipping an argument ending in `.go`.
A command whose directory cannot be placed ends the walk, as in `crossModuleCommands`.

`checkVerifyPackageMissing` reads each card's `Verify` value and the overview's `plan.Verify` chain.
The created directories for a card are the `Dir` of every `CardTargetDirs` entry with `DeleteOnly` false, over every card numbered at or below it; the overview chain uses the whole plan.
A `packageArgument` is a finding when its `Dir` is not a directory on disk under `worktreeRoot` (`os.Stat` plus `IsDir`) and is neither a created directory nor an ancestor of one (`.` is the ancestor of every directory).
A card's Verify is not checked once any card numbered at or below it has a `Move` group, and the overview chain is not checked when any card has one, since a Move's destination lives in Intent prose.
A directory an earlier card deletes is not reported; the card's own verify run catches it.
One finding per argument: `Check` `verify-package-missing`, `Card` the card's id or empty for the overview, `Ref` the argument as written, and a `Detail` naming the card's Verify or "the overview's `## verify:` section", the command and the argument, saying it names no directory in the worktree nor one a card up to that point creates, and ending "correct the Verify line".
When `externalTestPackageDir(Dir)` differs from `Dir` and that directory exists or is created, the detail adds "it is an external test package, whose tests live in `<corrected>`", `<corrected>` being the argument with its `_test` suffix removed (and its `/...` kept).
It runs under any `language:`.

`validate` calls it right after `checkVerifyModuleWide`, so `Validate` and `ValidateFormat` both report it, and validate.go's file comment lists `verify-package-missing (checkVerifyPackageMissing, verifymodule.go)` after `verify-module-wide`.
Dispatch already drops a finding against a completed card.
This check only refuses: a spelling behind a variable or `bash -c`, after an unplaceable `cd`, an import path, a module-wide pattern that trims to an existing directory, and a `.go` file argument pass.

`TestValidate_VerifyPackageMissing`, in `internal/planparser/validate_test.go` beside `TestValidate_VerifyModuleWide`, is one table over `ValidateFormat` against a `t.TempDir()` root holding `internal/x/x.go` and `nested/pkg/p.go`, collecting the `Card` of every `verify-package-missing` finding.
Rows that report:
- a card Verify `lyx gate test ./internal/x_test`, whose finding's `Ref` is `./internal/x_test` and whose detail also holds "tests live in `./internal/x`";
- the overview chain `go test ./nope`, attributed to no card;
- the overview chain `go build ./nope`;
- the overview chain `go vet ./nope`;
- the overview chain `go test ./nope/...`;
- a card Verify `go -C nested test ./nope`, resolved against the module;
- a card Verify `lyx gate test --tags integration ./nope`;
- card 1's Verify `go test ./internal/new` when only card 2 creates `internal/new/n.go`;
- a card Verify `go test ./nope` under `language: none`.
Rows that report nothing:
- `go test ./internal/x`;
- `lyx gate test -C nested --tags integration ./pkg`;
- a Verify on the same card that creates `internal/new2/y.go`;
- card 1 a `Create` of `internal/new/x.go#Foo`, then card 2's Verify `go test ./internal/new`;
- card 1 a `Custom` of `internal/new/x.go`, then card 2's Verify on `./internal/new`;
- card 1 a `Rename` pair whose New side is `internal/new/n.go`, then card 2's Verify on `./internal/new`; the row builds the group inline with `Pairs` set, as `TestCardTargetDirs`' Rename row does, since `cardOfType` sets `Refs` alone and `CardTargetDirs` reads a Rename group's `Pairs`;
- card 1 a `Create` of `brandnew/sub/x.go`, then card 2's Verify `go test ./brandnew`, an ancestor of a created directory;
- `go test ./...`;
- the import path `go test example.com/m/nope`;
- the file argument `go test ./nope/x_test.go`;
- `cd /abs && go test ./nope`;
- `go run ./nope`;
- card 1 a `Move` group, then card 2's Verify `go test ./nope`;
- the overview chain `go test ./nope` in a plan with a `Move` card.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/planparser` by one for the new top-level test.

Docs: `internal/planparser/doc.go`'s Verify chains section names `verify-package-missing` among the checks reading a chain through the one walk, and `contracts/specs/loom-plan-spec.md` gains row 42 after `fabric-vocabulary-name`, stating the rule, the created-directory set, the Move skip, the `_test` hint, the way forward, that it runs under any `language:` in both plan gates, `lyx loom validate-plan` and dispatch through `ValidateFormat`, and the static-spelling bound.

**ImpactSummary:** a plan whose Verify or overview chain names a relative package directory that neither exists nor is created fails both plan gates; `crossModuleCommands` keeps its verdicts through the shared chain walk.

**Commit:** 2: planparser: verify-package-missing refuses a Verify argument naming no directory
