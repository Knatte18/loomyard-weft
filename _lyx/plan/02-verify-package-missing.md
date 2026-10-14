# Card 2 — verify-package-missing

**Create:**
- `plan:internal/planparser#checkVerifyPackageMissing` -> `func checkVerifyPackageMissing(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser#missingPackageArg` -> `type missingPackageArg struct`
- `plan:internal/planparser#missingPackageArgs` -> `func missingPackageArgs(chain, worktreeRoot string, created []string) []missingPackageArg`
- `plan:internal/planparser#gateTestDirAndPackages` -> `func gateTestDirAndPackages(runDir string, fields []string) (commandDir string, packageArgs []string, ok bool)`
- `plan:internal/planparser#createdDirs` -> `func createdDirs(plan *Plan, through int) []string`
- `plan:internal/planparser_test#TestValidate_VerifyPackageMissing` -> `func TestValidate_VerifyPackageMissing(t *testing.T)`

**Edit:**
- `internal/planparser#validate`
- `internal/loomcli#TestGateParity_PlanGate`
- `internal/planparser/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#splitChain`
- `internal/planparser#isEnvAssignment`
- `internal/planparser#resolveRunDir`
- `internal/planparser#goCommandDirAndPackages`

**Intent:** Add the blocking plan check `verify-package-missing` (D3): an author-written Verify line whose relative package argument names no directory, such as `./internal/x_test`, is refused before implementation.

The new members live in `internal/planparser/verifymodule.go`, beside the two verify checks whose chain reading they share; the file's leading comment names the third check.

- `checkVerifyPackageMissing` reads each card's `**Verify:**` value and the overview's `## verify:` chain.
  A card's value is judged against `createdDirs(plan, card.Number)`, the overview's against `createdDirs(plan, <highest card number>)`.
  A card's value is skipped when any card numbered at or below it has a `Move` group, and the overview chain is skipped when any card has one, since a Move's destination lives in Intent prose.
  One finding per offending argument: `Check` `verify-package-missing`, `Card` the card's `N-<slug>` id (empty for the overview), `Ref` the argument as written, and a `Detail` naming the card (or "the overview's `## verify:` section"), the command and the argument, ending in the way forward "correct the Verify line".
  When the argument's directory ends in `_test` and `externalTestUnitDir` of it exists on disk or is created, the detail also says the argument is an external test package and names the corrected argument, for example `./internal/x`.
- `createdDirs` returns the `Dir` of every `CardTargetDirs` entry with `DeleteOnly` false, over every card numbered at or below `through`: a path or glyph target of any non-Delete group, member glyphs and Custom groups included, and a Rename pair's New side.
- `missingPackageArgs` scans one chain the way `crossModuleCommands` does: `splitChain`, leading `isEnvAssignment` fields skipped, a run directory starting at `.` that follows a literal `cd <dir>` after a sequencing separator through `resolveRunDir`, and a scan that ends at a `cd` it cannot place (wrong arity, a pipeline stage or after `||`, or `resolveRunDir` refusing it).
  It reads two command shapes.
  For `go test`, `go build` and `go vet` it takes the command directory and relative arguments from `goCommandDirAndPackages`, and checks the subcommand itself, since that helper does not; an unplaceable `-C` ends the scan.
  For `lyx gate test` it takes them from `gateTestDirAndPackages`, which reads the fields after `test`: `-C <dir>`, `-C=<dir>`, `--dir <dir>` and `--dir=<dir>` set the command directory through `resolveRunDir` (ok false when it cannot be placed), `--tags <v>` and `--tags=<v>` are skipped with their value, reading stops at `--`, and each remaining field spelled `.`, `..`, `./…` or `../…` is a package argument.
  Each package argument not ending in `.go` is joined onto the command directory with a trailing `/...` removed; it is missing when that directory neither exists on disk under `worktreeRoot` nor is, or is an ancestor of, an entry of `created`.
  `.` resolves to the root and always exists, so `./...` passes.
  `missingPackageArg` carries the command text, the argument and the corrected argument (empty when there is none).
- `validate` appends `checkVerifyPackageMissing(plan, worktreeRoot)` right after `checkVerifyModuleWide`, so both `ValidateFormat` and `Validate` report it; validate.go's leading comment adds `verify-package-missing (checkVerifyPackageMissing, verifymodule.go)` to its check list.
  Webster dispatch reaches it through `ValidateFormat` and already drops a finding against a completed card.
  It runs under any `language:`.

Bound: it reads static spellings only, so an argument behind a variable or `bash -c`, after an unplaceable `cd`, an import path, a module-wide pattern resolving to an existing directory and a `.go` file argument all pass.
A directory an earlier card deletes is not reported; that card's own verify run catches it.
It only refuses and removes no guard.

Tests:

- New `TestValidate_VerifyPackageMissing` in `validate_test.go`, table-driven like `TestValidate_VerifyNestedModule`, over a `t.TempDir()` root materialized per row with `materializeFiles`, filtering `ValidateFormat`'s findings to `verify-package-missing` and comparing the `Card` list and, where set, a detail substring.
  Rows that fire: a card Verify `lyx gate test ./internal/x_test` with `internal/x/x.go` on disk, whose detail names `./internal/x`;
  an overview `go test ./nope`, with an empty `Card`;
  `go -C sub test ./internal/x` with `internal/x/x.go` on disk and no `sub/internal/x`, resolved against `sub`;
  `lyx gate test -C sub ./internal/x` on the same disk;
  a card 1 Verify `go test ./internal/new` where only card 2 creates `internal/new/x.go`;
  `go vet ./nope` and `go build ./nope`, one row each, so the subcommand filter keeps `vet` and `build`;
  a card 2 Verify `lyx gate test ./internal/new_test` where card 1 creates `internal/new/x.go` and nothing is on disk, whose detail names `./internal/new`, the hint for a created directory.
  Rows that stay silent: an existing directory;
  a directory the card's own Create group targets with the path `internal/new/x.go`, and the same on an earlier card;
  a Create card targeting the member glyph `internal/new/x.go#Foo` followed by a card Verify `go test ./internal/new`;
  a directory on disk, `internal/gone/g.go`, that an earlier Delete-only card removes, followed by a Verify `go test ./internal/gone`;
  a Custom card targeting `internal/new/x.go` followed by the same Verify;
  a Rename card whose pair's New side is `plan:internal/new#Moved` followed by the same Verify;
  `go test ./internal` when a card creates `internal/new/x.go` and no `internal` exists on disk, an ancestor of a created directory;
  `go test ./...`;
  an import path `go test github.com/x/y`;
  a `.go` file argument `go test ./nope/a.go`;
  `cd /abs && go test ./nope`, after an unplaceable `cd`;
  `go run ./nope`, not a test, build or vet;
  a card Verify `go test ./nope` on a card after a card with a `Move` group, and on the Move card itself.
- `TestValidate_GoldenFixture_ZeroFindings` stays green unedited: the fixture's card Verify and overview arguments resolve to directories it materializes, and its overview follows a Move card.
- `TestGateParity_PlanGate` gains a row `VerifyPackageMissing`: `verifySectionPlanFixture` with `go test ./nope` in the verify section, where both the gate and the verb report stuck with `verify-package-missing`.
- `cmd/lyx/testdata/test-budget.yaml`'s `internal/planparser` row rises from 92 to 93 for the new top-level test.

Docs in this commit:

- `contracts/specs/loom-plan-spec.md` gains check row 42, `verify-package-missing`, after row 41: what it reads, the two command shapes, the `/...` trim and `.go` skip, the created-directory rule, the Move skip, the `_test` replacement hint, the way forward "correct the Verify line", that it runs under any `language:` and is reported by both plan gates, `lyx loom validate-plan` and dispatch through `ValidateFormat`, and the bound above.
- `internal/planparser/doc.go`'s "Verify chains" section names `verify-package-missing` as the third reader of the shared splitter.

**ImpactSummary:** Adds one blocking check to planparser's shared validate list, so every plan gate, `lyx loom validate-plan` and dispatch report it; existing plans whose Verify arguments name real or created directories, the golden fixture included, are unaffected.

**Commit:** `2: plan check refuses a Verify argument that names no directory`
