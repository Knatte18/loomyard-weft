# Card 2 — verify-package-missing

**Create:**
- `internal/planparser/verifypackage.go`
- `plan:internal/planparser#checkVerifyPackageMissing` -> `func checkVerifyPackageMissing(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser#relativePackageArgs` -> `func relativePackageArgs(chain string) []packageArg`
- `plan:internal/planparser#packageArg` -> `type packageArg struct`
- `plan:internal/planparser#gateTestDirAndPackages` -> `func gateTestDirAndPackages(runDir string, fields []string) (commandDir string, packageArgs []string, ok bool)`
- `plan:internal/planparser_test#TestValidate_VerifyPackageMissing` -> `func TestValidate_VerifyPackageMissing(t *testing.T)`

**Edit:**
- `internal/planparser#validate`
- `internal/planparser/doc.go`
- `internal/loomcli#TestGateParity_PlanGate`
- `contracts/specs/loom-plan-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#splitChain`
- `internal/planparser#isEnvAssignment`
- `internal/planparser#resolveRunDir`
- `internal/planparser#goCommandDirAndPackages`
- `internal/planparser#CardTargetDirs`
- `internal/planparser#pathExistsOnDisk`
- `plan:internal/planparser#goPackageDir`

**Intent:** D3: a new blocking plan check, `verify-package-missing`, over author-written verify chains, in a new file `internal/planparser/verifypackage.go` (file comment naming the check).

`checkVerifyPackageMissing` reads each card's `Verify` value and the overview's `plan.Verify` chain.
`relativePackageArgs` walks one chain the way `crossModuleCommands` does: `splitChain`, leading `isEnvAssignment` fields skipped, a literal `cd <dir>` followed through `resolveRunDir` only after a sequencing separator, and a `cd` in a pipeline stage, after `||`, or one `resolveRunDir` cannot place ends the scan (an argument after an unplaceable `cd` passes).
For a `go` command it calls `goCommandDirAndPackages`, and only for the `test`, `build` and `vet` subcommands; for a `lyx gate test` command it calls `gateTestDirAndPackages`.
`packageArg` carries the argument as written and its directory: the argument joined onto the command's directory with any trailing `/...` removed (`path.Join`).
An argument already ending in `/...` with a non-empty remainder is checked by its remainder; `./...` therefore joins to the command directory itself and always passes.

`goCommandDirAndPackages` is read, not changed: its package list is subcommand-blind and keeps relative arguments ending in `.go`, so `relativePackageArgs` drops every argument ending in `.go` itself and reads the subcommand from the fields after the `-C` flags to restrict the check to `test`, `build` and `vet`.

`gateTestDirAndPackages` reads the fields after `lyx gate test`: `-C <dir>`, `-C=<dir>`, `--dir <dir>` and `--dir=<dir>` set the command directory through `resolveRunDir` (ok false when it cannot be placed, which ends the scan like `cd`), `--tags <v>` and `--tags=<v>` are skipped with their value, the fields stop at `--`, and every other field spelled relative (`.`, `..`, `./…`, `../…`) and not ending in `.go` is a package argument.

The created directories for a card numbered N are the `Dir` of every `CardTargetDirs` entry with `DeleteOnly` false over every card numbered at or below N; for the overview chain, over the whole plan.
An argument is a finding when its directory neither exists on disk as a directory under `worktreeRoot` (`os.Stat` and `IsDir`) nor equals, nor is an ancestor of, a created directory.
A card's Verify is not checked when that card or any earlier card carries a Move group, and the overview chain is not checked when any card carries one, since a Move's destination lives in Intent prose.
A directory an earlier card deletes is not reported.

The finding is `Check: "verify-package-missing"`, `Card: cardID(c)` (empty for the overview), `Ref` the argument, and a Detail that names the card (or "the overview's `## verify:` section") and the argument, says no such directory exists or is created by this or an earlier card, and gives the way forward: correct the Verify line to name an existing package directory.
When the argument's directory ends in `_test` and `goPackageDir` of it exists on disk or is created, the Detail says it is an external test package whose tests live in that directory and names the corrected argument, for example `./internal/x`.

`validate` appends `checkVerifyPackageMissing(plan, worktreeRoot)` right after `checkVerifyModuleWide`, so it runs in `Validate` and `ValidateFormat` and is blocking; dispatch already drops a finding against a completed card.
The `validate.go` file comment that enumerates the check IDs gains `verify-package-missing (checkVerifyPackageMissing, verifypackage.go)` after `verify-module-wide`.

The check only refuses: an argument behind a variable or `bash -c`, after an unplaceable `cd`, an import path, an absolute path, a module-wide pattern or a `.go` file argument passes.

Tests:
- `TestValidate_VerifyPackageMissing`, table-driven over `ValidateFormat` with a `t.TempDir()` root holding `internal/x/x.go` and `nested/go.mod` plus `nested/pkg/p.go` (built with `materializeFiles`), collecting the `Card` and `Detail` of each `verify-package-missing` finding.
  A hint row asserts the Detail holds `external test package`, text only the hint carries, since the echoed argument already contains its corrected form; every other reporting row asserts the Detail does not hold it.
  Rows that report: a card Verify `lyx gate test ./internal/x_test`, a hint row (the trimmed directory exists on disk); card 1 creating `internal/y/y.go#Foo` and card 2's Verify `lyx gate test ./internal/y_test`, a hint row (the trimmed directory is created); `lyx gate test ./internal/x -- ./nope` reports nothing while `lyx gate test ./internal/x ./nope` reports, pinning the stop at `--`; overview chain `go test ./nope` (Card empty); `go -C nested test ./missing` resolved against `nested`; `lyx gate test -C nested ./missing`; card 1 Verify `go test ./internal/new` where only card 2 creates `internal/new/x.go#Foo`; `go vet ./nope` and `go build ./nope`; one finding per bad argument in `go test ./nope ./internal/x`.
  Rows that report nothing: an existing directory (`go test ./internal/x`); a directory the same card creates a target in; a Create card targeting `internal/new/x.go#Foo` followed by a card whose Verify is `go test ./internal/new`; a Custom card targeting `internal/new/x.go` followed by a Verify on `./internal/new`; a Rename card whose pair's New side is `plan:internal/new/n.go#New` followed by a Verify on `./internal/new`; `go test ./...`; `go test github.com/x/y`; `go test ./internal/x/x_test.go`; `cd "$DIR" && go test ./nope`; a Verify on a card after a Move card; a Verify on the Move card itself; a card creating `internal/new/sub/x.go#Foo` followed by a Verify on `./internal/new`, with `internal/new` absent on disk (the ancestor rule); `go list ./nope` and `go run ./nope` (outside the subcommand filter); `lyx gate test --tags integration ./internal/x -- -run X`.
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/planparser` from 92 to 93 for the new top-level test.
- `TestGateParity_PlanGate` gains a `VerifyPackageMissing` row whose `verifySectionPlanFixture` verify body is `go test ./nope`, with `wantGate` and `wantCLI` both stuck and `wantFinding: "verify-package-missing"`.

Docs in the same commit:
- `internal/planparser/doc.go`'s `# Verify chains` section names `verify-package-missing` as the third reader of the splitter, and states what it checks and its bound.
- `contracts/specs/loom-plan-spec.md` gains check `42. verify-package-missing` in the validation-checks list, stating the trigger, the created-directory rule, the Move and Delete bounds, the `_test` hint, the way forward (correct the Verify line), that it runs under any `language:` in both `Validate` and `ValidateFormat`, so both plan gates, `lyx loom validate-plan` and dispatch report it, and that it only refuses and misses the static-spelling cases above.

**ImpactSummary:** one new blocking check appended to `validate`, reaching both plan gates, `lyx loom validate-plan` and dispatch; the shared chain helpers are only read, so `verify-nested-module` and `verify-module-wide` are unchanged.

**Commit:** `2: plan check verify-package-missing for a Verify argument that names no directory`
