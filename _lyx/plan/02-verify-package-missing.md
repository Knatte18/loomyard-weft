# Card 2 — Plan check verify-package-missing

**Create:**
- `plan:internal/planparser#verifyPackageArg` -> `type verifyPackageArg struct`
- `plan:internal/planparser#verifyPackageArgs` -> `func verifyPackageArgs(chain string) []verifyPackageArg`
- `plan:internal/planparser#gateCommandDirAndPackages` -> `func gateCommandDirAndPackages(runDir string, fields []string) (commandDir string, packageArgs []string, ok bool)`
- `plan:internal/planparser#checkVerifyPackageMissing` -> `func checkVerifyPackageMissing(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser_test#TestValidate_VerifyPackageMissing` -> `func TestValidate_VerifyPackageMissing(t *testing.T)`

**Edit:**
- `internal/planparser#validate`
- `internal/planparser/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#splitChain`
- `internal/planparser#isEnvAssignment`
- `internal/planparser#resolveRunDir`
- `internal/planparser#goCommandDirAndPackages`
- `plan:internal/planparser#goPackageUnit`

**Intent:**
D3: after card 1, an author-written Verify line or the overview's verify chain is the remaining route for a package argument that names no directory.
The new members live in `internal/planparser/verifymodule.go`, beside the two verify checks whose chain parsing they share.

`verifyPackageArg` is one relative package argument found in a chain: `Command` (the command's text), `Arg` (the argument as written) and `Dir` (the worktree-relative directory it names: the argument joined onto the command's directory, any trailing `/...` removed).

`verifyPackageArgs` walks a chain the way `crossModuleCommands` does: `splitChain`, leading environment assignments skipped, the run directory starting at `.` and following a literal `cd <dir>` after a sequencing separator through `resolveRunDir`, and ending the scan at a `cd` in a pipeline stage, after `||`, or to a spelling `resolveRunDir` cannot place.
It reads two command shapes:
- `go …`: through `goCommandDirAndPackages`; a `-C` it cannot place ends the scan.
- `lyx gate test …`: through `gateCommandDirAndPackages`, which reads the fields after `test`: `-C <dir>`, `-C=<dir>`, `--dir <dir>` and `--dir=<dir>` set the command directory through `resolveRunDir` (ok false when it cannot place it, which ends the scan), `--tags <v>` and `--tags=<v>` take their value, `--` ends the package arguments, any other flag ends the command's arguments, and a field spelled `.`, `..`, `./…` or `../…` is a package argument.
Of each command's relative arguments it keeps those not ending in `.go`; an import path, an absolute path and an argument behind a variable or `bash -c` never reach it.
An argument whose join leaves the worktree root, such as `../x` from the root, is unplaceable and skipped, the bound `resolveRunDir` applies to `cd` and `-C`.
`go` commands other than `test`, `build` and `vet` are not read.

`checkVerifyPackageMissing` reports one `verify-package-missing` finding per argument whose `Dir` neither exists as a directory under `worktreeRoot` nor is, or is an ancestor of, a created directory.
The created directories of a card's Verify are the `Dir` of every `CardTargetDirs` entry with `DeleteOnly` false over the cards numbered at or below that card; for the overview's `## verify:` chain, over every card.
A card's Verify on or after the first card carrying a `Move` group is not read, and neither is the overview chain when any card carries one, since a Move's destination lives in Intent prose.
A card finding is attributed to the card, with the detail `card N Verify: `<command>` names <arg>, but <dir> is no directory in the worktree and no card up to N creates one; way forward: correct the Verify line`.
An overview finding has no card, with the detail `the overview's `## verify:` section: `<command>` names <arg>, but <dir> is no directory in the worktree and no card creates one; way forward: correct the verify line`.
When `goPackageUnit(dir)` differs from `dir` and that trimmed directory exists or is created, the detail adds `; <arg> is an external test package, whose tests live in <trimmed>; name ./<trimmed>` with the argument's own spelling trimmed the same way.
It runs under any `language:`, like `verify-nested-module`.

`validate` dispatches it right after `checkVerifyModuleWide`, so `Validate` and `ValidateFormat` both report it and it blocks; dispatch already drops a finding against a completed card.
`validate.go`'s header comment lists `verify-package-missing (checkVerifyPackageMissing, verifymodule.go)` after `verify-module-wide`, and `verifymodule.go`'s own header comment names the third check.

`TestValidate_VerifyPackageMissing`, in `validate_test.go`, is a table shaped like `TestValidate_VerifyNestedModule`: each row builds cards and an overview chain, runs `ValidateFormat` against a root that `materializeFiles` populates with `internal/x/x.go`, `nested/go.mod` and `nested/pkg/p.go`, and compares the cards of the `verify-package-missing` findings, plus a detail substring where the row names one.
Rows that report:
- a card Verify `lyx gate test ./internal/x_test`, with the detail holding `name ./internal/x`;
- a Create card targeting `internal/made/m.go` whose Verify runs `lyx gate test ./internal/made_test`, with the detail holding `name ./internal/made`, the hint from a created directory absent on disk;
- the overview chain `go test ./nope`, attributed to no card;
- card Verifies `go build ./nope` and `go vet ./nope`, one finding each;
- `go test ./pkg`, which from the root names the absent `pkg`;
- `lyx gate test -C nested ./nope`, resolved against `nested`;
- card 1's Verify `go test ./internal/new` where card 2 creates `internal/new/x.go`.
Rows that report nothing:
- `go test ./internal/x` and `lyx gate test --tags integration ./internal/x`;
- `go -C nested test ./pkg` and `lyx gate test -C nested ./pkg`;
- a Create card targeting `internal/new/x.go#Foo` whose own Verify, and a later card's Verify, run `go test ./internal/new`;
- a Custom card targeting `internal/new/x.go`, then a card whose Verify runs `go test ./internal/new`;
- a Rename card whose pair's New side is `plan:internal/new/n.go#New`, then a card whose Verify runs `go test ./internal/new`;
- a Create card targeting `newroot/sub/s.go` whose Verify runs `go test ./newroot/...`, the ancestor of a created directory;
- `go test ./...`, `go test github.com/Knatte18/loomyard/internal/nope`, `go test ./internal/nope/x_test.go`;
- `cd /abs && go test ./nope`, an unplaceable `cd`;
- `go test ../outside`, a join that leaves the worktree root;
- `go run ./nope`, a go subcommand the check does not read;
- a Move card whose own Verify runs `go test ./nope`, a later card whose Verify runs the same, and the overview chain `go test ./nope` in that plan.

The test is a new top-level test, so `cmd/lyx/testdata/test-budget.yaml`'s `internal/planparser` row rises from 92 to 93.
`contracts/specs/loom-plan-spec.md` gains entry `42. verify-package-missing` after `fabric-vocabulary-name`, in the shape of entry 38: what it reports, the chain reading it shares with `verify-nested-module`, the two command shapes, the created-directory rule, the external-test hint, the way forward, that both plan gates, `lyx loom validate-plan` and dispatch report it through `ValidateFormat`, and its bound: static spellings only, nothing read on or after a Move card, and a directory an earlier card deletes not reported since the card's own run catches it.
`internal/planparser/doc.go`'s Verify chains section names `verify-package-missing` beside the two checks that share the splitter.

**ImpactSummary:** a new blocking plan check over card Verify lines and the overview chain; no existing check, caller or fixture changes, and the golden fixture's Verify lines name directories it materializes.

**Commit:** 2: add the verify-package-missing plan check
