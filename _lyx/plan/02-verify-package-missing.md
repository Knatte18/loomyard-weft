# Card 2 — verify-package-missing

**Create:**
- `plan:internal/planparser#checkVerifyPackageMissing` -> `func checkVerifyPackageMissing(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser#missingPackageArg` -> `type missingPackageArg struct`
- `plan:internal/planparser#missingPackageArgs` -> `func missingPackageArgs(chain, worktreeRoot string, created []string) []missingPackageArg`
- `plan:internal/planparser#gateTestDirAndPackages` -> `func gateTestDirAndPackages(runDir string, fields []string) (commandDir string, packageArgs []string, ok bool)`
- `plan:internal/planparser#createdDirs` -> `func createdDirs(plan *Plan, through int) []string`

**Edit:**
- `internal/planparser#goCommandDirAndPackages` -> `func goCommandDirAndPackages(runDir string, fields []string) (commandDir, subcommand string, packageArgs []string, ok bool)`
- `internal/planparser#crossModuleCommands`
- `internal/planparser#validate`
- `internal/planparser/validate_test.go`
- `internal/planparser/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D3: a new blocking plan check, `verify-package-missing`, refuses an author-written Verify or `## verify:` command whose relative package argument names no directory.

Every new symbol lives in a new file, `internal/planparser/verifypackage.go`, whose header comment says what it holds, like `verifymodule.go`'s.

`goCommandDirAndPackages` re-signs to also return the subcommand, `fields[i]`, which it already reads; it returns "" with the existing early returns.
Its one caller, `crossModuleCommands`, discards it.

`gateTestDirAndPackages` reads the fields after `lyx gate test`: `-C <dir>`, `-C=<dir>`, `--dir <dir>` and `--dir=<dir>` set the command directory through `resolveRunDir` (ok false when it cannot be placed), `--tags <tags>` and `--tags=<tags>` skip their value, and reading stops at `--`.
Any other field spelled relative (`.`, `..`, `./…`, `../…`) is a package argument, wherever it sits among the flags.

`missingPackageArgs` walks one chain exactly as `crossModuleCommands` does: `splitChain`, leading `isEnvAssignment` fields skipped, a literal `cd <dir>` followed through `resolveRunDir`, and the scan ended at a `cd` in a pipeline stage, after `||`, or one that cannot be placed, and at a `-C` that cannot be placed.
It reads a `go` command through `goCommandDirAndPackages` when the subcommand is `test`, `build` or `vet`, and a `lyx gate test` command through `gateTestDirAndPackages`.
For each package argument not ending in `.go`, it removes a trailing `/...` and joins the rest onto the command's directory with `path.Join`; a result outside the worktree root (`..` or `../…`) passes.
The argument is missing when that directory is not a directory on disk under `worktreeRoot` and is neither equal to nor an ancestor of an entry of `created`.
A `missingPackageArg` holds the command text, the argument as written and a hint: when the trimmed argument ends in `_test` and its `_test`-less spelling joined onto the command directory exists or is created by that same rule, the hint is the argument with `_test` removed (a trailing `/...` kept), else empty.

`createdDirs` returns the `Dir` of every `CardTargetDirs` entry with `DeleteOnly` false over the cards numbered at or below `through`, so a path or glyph target of any non-Delete group, Custom groups and member glyphs included, and a Rename pair's New side all count; after card 1 an `internal/x_test` target counts as `internal/x`.

`checkVerifyPackageMissing` checks each card with a non-empty `Verify` against `createdDirs(plan, card.Number)`, and the overview's `plan.Verify` against the whole plan's.
A card's Verify is skipped when that card or an earlier one holds a `CardTypeMove` group, and the overview chain when any card does, since a Move's destination lives in Intent prose.
Each missing argument is one finding: `Check` `verify-package-missing`, `Card` the card's `cardID` (empty for the overview), `Ref` the argument, and a `Detail` such as "card 3 Verify: `lyx gate test ./internal/x_test` names ./internal/x_test, which is no directory on disk and none that card 3 or an earlier card creates; correct the Verify line", or for the overview "the overview's `## verify:` section: … no directory on disk and none the plan creates; correct the Verify line".
With a hint, the detail adds "; ./internal/x_test is an external test package whose tests live in ./internal/x: pass ./internal/x".
The way forward is a plan edit any writer can make, so `PATTERN-no-denied-recovery` holds.
It runs under any `language:`.

`validate` appends it right after `checkVerifyModuleWide`, so both `Validate` and `ValidateFormat` report it and it blocks.
`validate.go`'s leading comment, which lists every check ID and sits outside `validate`'s span, lists the new ID beside `verify-module-wide`; the card names `validate` rather than the file, since a file beside its member is refused as `redundant-file-target`.
Dispatch already drops a finding against a completed card.

Bound: it reads static spellings only, so an argument behind a variable, inside `bash -c`, or after an unplaceable `cd` passes, as do an import path, a module-wide pattern whose stem exists and a `.go` file argument.
A directory an earlier card deletes is not reported; that card's own verify run catches it.
The check only refuses and removes no guard.

Tests: a new `TestValidate_VerifyPackageMissing` in `internal/planparser/validate_test.go`, table-driven like `TestValidate_VerifyNestedModule`, over a root that `materializeFiles` gives `internal/x/x.go`, `m/go.mod` and `m/pkg/p.go`.
Each row runs `planparser.ValidateFormat` and compares the `(Card, Ref)` pairs of its `verify-package-missing` findings to the row's want, and every finding's detail must contain `correct the Verify line`.
Rows that report:
- a card Verify `lyx gate test ./internal/x_test`, whose detail also contains `./internal/x` as the replacement;
- the overview chain `go test ./nope`, with an empty card;
- `go build ./nope` and `go vet ./nope`, one row each;
- `go -C m test ./absent`, resolved against the module;
- `lyx gate test -C m ./absent`, `lyx gate test -C=m ./absent`, `lyx gate test --dir m ./absent` and `lyx gate test --dir=m ./absent`, one row each;
- `lyx gate test --tags integration ./nope`, where the tag value is skipped and `./nope` still read;
- a card 1 Verify `go test ./internal/new` where only card 2 creates `internal/new/n.go`.

Rows that report nothing:
- `go test ./internal/x` on disk;
- a card whose own Create group targets `internal/new/n.go#` with a Verify on `./internal/new`;
- a card 2 Verify on `./internal/new` after a card 1 that creates it;
- a Create card targeting `internal/new/x.go#Foo` followed by a Verify on `./internal/new`;
- a Custom card targeting `internal/new/x.go` followed by a Verify on `./internal/new`;
- a Rename card whose group carries `Pairs: []planparser.MovePair{{Old: "internal/old/o.go#Old", New: "plan:internal/new/n.go#New"}}`, built as `TestCardTargetDirs`' Rename row builds it since `cardOfType` sets no `Pairs`, followed by a Verify on `./internal/new`;
- `go test ./newroot` where a card creates `newroot/sub/s.go`, since the directory is an ancestor of a created one;
- `go test ./...`, `go test example.com/x` and `go test ./internal/absent/a_test.go`;
- `go -C m test ./pkg` and `lyx gate test -C m ./pkg`;
- `lyx gate test ./internal/x -- ./nope`, where reading stops at `--`;
- `go run ./nope`, a subcommand the check does not read;
- `cd $DIR && go test ./nope`, after an unplaceable `cd`;
- a card 2 Verify `go test ./nope` after a card 1 with a Move group, and a Verify `go test ./nope` on a card with a Move group itself.

The test sits in the external `planparser_test` package for its `validCard`, `cardOfType` and `materializeFiles` helpers, so the card names the file rather than a `plan:internal/planparser_test#…` handle (see the overview's Shared Decisions).
The new top-level test raises `internal/planparser`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.
The golden fixture's `go test ./internal/boardcli` and `./internal/boardcli/...` lines resolve against its materialized tree, so `TestValidate_GoldenFixture_ZeroFindings` stays at zero findings.

Docs: `contracts/specs/loom-plan-spec.md`'s validation-checks list gains item 42, `verify-package-missing`, stating the rule, the created-directory set, the `_test` hint, the way forward, its place after `verify-module-wide` in `Validate` and `ValidateFormat`, that both plan gates, `lyx loom validate-plan` and dispatch report it, and the bound above.
`internal/planparser/doc.go`'s "Verify chains" section names it beside `verify-nested-module` and `verify-module-wide` as a reader of the one splitter.

**ImpactSummary:** one new check appended to `validate` and one re-signed private parser with a single in-package caller; every existing plan fixture's verify lines name directories its tree holds or `./...`, so only plans with a non-directory argument gain a blocking finding.

**Commit:** 2: Plan check verify-package-missing refuses a Verify argument that names no directory
