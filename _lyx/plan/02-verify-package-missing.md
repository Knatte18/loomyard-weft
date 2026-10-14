# Card 2 — verify-package-missing

**Create:**
- `internal/planparser/verifypackage.go`
- `plan:internal/planparser#checkVerifyPackageMissing` -> `func checkVerifyPackageMissing(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser#missingPackageArgs` -> `func missingPackageArgs(chain string, exists func(dir string) bool) []string`
- `plan:internal/planparser_test#TestValidate_VerifyPackageMissing` -> `func TestValidate_VerifyPackageMissing(t *testing.T)`

**Edit:**
- `internal/planparser/validate.go`
- `internal/planparser/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#splitChain`
- `internal/planparser#isEnvAssignment`
- `internal/planparser#resolveRunDir`
- `internal/planparser#goCommandDirAndPackages`
- `internal/planparser#CardTargetDirs`

**Intent:** D3: a new blocking plan check, `verify-package-missing`, catches an author-written gate or go command whose relative package argument names no directory, the route left for a bad path such as `./internal/x_test` after card 1.

`checkVerifyPackageMissing` reads each card's `Verify` value and the overview's `Plan.Verify` chain.
`missingPackageArgs` walks one chain with the shape `crossModuleCommands` uses: `splitChain`, leading `isEnvAssignment` fields skipped, a literal `cd <dir>` followed through `resolveRunDir` only after a sequencing separator, and the scan ended at a `cd` in a pipeline stage, after `||`, or to a spelling `resolveRunDir` refuses.
It reads two command shapes:
- `go` with the subcommand `test`, `build` or `vet`, its directory and relative package arguments from `goCommandDirAndPackages` (an unplaceable `-C` ends the scan); any other subcommand is skipped.
- `lyx gate test`, whose `-C <dir>`, `-C=<dir>`, `--dir <dir>` or `--dir=<dir>` sets its directory through `resolveRunDir`, whose `--tags <v>` or `--tags=<v>` value is skipped, and whose arguments stop at `--`.
A package argument is `.`, `..` or one opening with `./` or `../`, not ending in `.go`; it is joined onto the command's directory with any trailing `/...` removed, and reported when `exists` says no for that directory.
`missingPackageArgs` returns the offending arguments as written, in order.

For a card numbered N, `exists` holds for a directory that is on disk under `worktreeRoot` or that equals, or is an ancestor of, the `Dir` of a `CardTargetDirs` entry with `DeleteOnly` false on any card numbered at or below N; the overview chain counts every card.
So a path or glyph target of every non-Delete group, member glyphs and Custom groups included, and a Rename pair's New side each create their directory.
A Verify on a card numbered at or after the first card carrying a `Move` group is not checked, and the overview chain is not checked when any card carries one, since a Move's destination lives in Intent prose.
A directory an earlier card deletes is not reported; the card's own verify run catches it.

One finding per argument: `Check` `verify-package-missing`, `Card` the card's id or empty for the overview, `Ref` the argument, and a `Detail` naming the card (or the overview's `## verify:` section) and the argument, with the way forward to correct the Verify line.
When the argument ends in `_test` and its trimmed directory exists by the same rule, the detail names it as an external test package whose tests live in the trimmed directory and gives the corrected argument, for example `./internal/x`.
It runs under any `language:`, and `validate` appends it right after `checkVerifyModuleWide`, so `Validate`, `ValidateFormat`, both plan gates, `lyx loom validate-plan` and dispatch all report it; dispatch already drops a finding against a completed card.

Bound: it reads static spellings only, so an argument behind a variable, inside `bash -c`, after an unplaceable `cd`, an import path, an absolute path, a module-wide pattern resolving to an existing directory and a `.go` file argument pass; it only refuses and removes no guard, and webster's gates still run every command.

`validate.go` is listed whole because its package comment, outside every member, lists every check ID and gains `verify-package-missing (checkVerifyPackageMissing, verifypackage.go)`, beside the one-line dispatch addition in `validate`.
`internal/planparser/doc.go`'s Verify chains section names the third reader of the splitter, and `contracts/specs/loom-plan-spec.md` gains item 42, `verify-package-missing`, in the style of item 38: what it reads, the created-directory rule, the Move and Delete bounds, the `_test` hint, the way forward and the gates that report it.

`TestValidate_VerifyPackageMissing` is table-driven over `ValidateFormat` on a `t.TempDir()` root with `materializeFiles` creating `internal/x/x.go` and `mod/go.mod`, collecting `(Card, Ref)` of the check's findings, and asserting the detail of the `_test` row holds `./internal/x`.
Reported: a card Verify `lyx gate test ./internal/x_test`; the overview chain `go test ./nope`; `go build ./nope`; `go -C mod test ./nope` and `lyx gate test -C mod ./x`, resolved against `mod`; `lyx gate test --tags integration ./nope`, whose `--tags` value is skipped and the argument after it read; a card 1 Verify `go test ./internal/new` whose directory only card 2 creates.
Silent: `go test ./internal/x`; a card whose own Edit target is `internal/new/x.go` carrying the Verify `go test ./internal/new`, the same-card case; a Create card targeting `internal/new/x.go#Foo` then a Verify on `./internal/new`; a Custom card targeting `internal/new/x.go` then a Verify on `./internal/new`; a Rename card whose pair's New side is `plan:internal/new/n.go#New` then a Verify on `./internal/new`; a Create card targeting `fresh/new/x.go` then a Verify on the ancestor `./fresh`, which exists nowhere on disk; `go test ./...`; an import path `example.com/m/pkg`; `go vet ./internal/new/x.go`; `cd /abs && go test ./nope`; `lyx gate test ./internal/x -- ./nope`, whose field after `--` is not read; a Verify `go test ./nope` on a card carrying a Move group, and the same on a card after it; the overview chain `go test ./nope` in a plan with a Move card; `go run ./nope`, whose subcommand is not read.
The new top-level test raises `internal/planparser`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.

**ImpactSummary:** Adds one blocking plan check over Verify lines and the overview chain; existing tests filter findings by check ID, and the loomcli parity fixtures that now also trip it already expect a stuck verdict.

**Commit:** `2: Plan check verify-package-missing refuses a Verify package that names no directory`
