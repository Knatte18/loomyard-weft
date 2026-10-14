# Card 2 — verify-package-missing

**Create:**
- `plan:internal/planparser#checkVerifyPackageMissing` -> `func checkVerifyPackageMissing(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser#relativePackageDirs` -> `func relativePackageDirs(chain string) []string`
- `plan:internal/planparser#gateTestDirAndPackages` -> `func gateTestDirAndPackages(runDir string, fields []string) (commandDir string, packageArgs []string, ok bool)`
- `plan:internal/planparser_test#TestValidate_VerifyPackageMissing` -> `func TestValidate_VerifyPackageMissing(t *testing.T)`

**Edit:**
- `internal/planparser#validate`
- `internal/loomcli#TestGateParity_PlanGate`
- `internal/planparser/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/planparser#goPackageDir`
- `internal/planparser#splitChain`
- `internal/planparser#isEnvAssignment`
- `internal/planparser#resolveRunDir`
- `internal/planparser#goCommandDirAndPackages`
- `internal/planparser#CardTargetDirs`

**Intent:** D3: a new blocking plan check, `verify-package-missing`, refuses an author-written Verify package argument that names no directory.

The new members live in `internal/planparser/verifymodule.go`, beside `verify-nested-module`, and reuse its shell reading; the file's head comment, which names the two checks it holds, names `verify-package-missing` as the third.
`relativePackageDirs` scans one chain with `splitChain`, skipping leading environment assignments with `isEnvAssignment` and tracking a literal `cd <dir>` with `resolveRunDir` exactly as `crossModuleCommands` does: a `cd` in a pipeline stage, after `||`, to an absolute path or to an unplaceable spelling ends the scan.
For a `go` command it reads `goCommandDirAndPackages`, and acts only on `test`, `build` and `vet`; for `lyx gate test` it reads `gateTestDirAndPackages`.
`gateTestDirAndPackages` reads the fields after `lyx gate test`: `-C <dir>`, `-C=<dir>`, `--dir <dir>` and `--dir=<dir>` set the command directory through `resolveRunDir` (ok false when it cannot be placed, which ends the scan); `--tags <tags>` and `--tags=<tags>` are skipped; reading stops at `--`; every other field spelled `.`, `..`, `./…` or `../…` is a package argument.
Each package argument not ending in `.go` is joined onto the command's directory with any trailing `/...` removed, and the joined worktree-relative directory is returned.

`checkVerifyPackageMissing` reads each card's `**Verify:**` value and the overview's `## verify:` chain.
The created directories of a card's Verify are the `Dir` of every `CardTargetDirs` entry with `DeleteOnly` false over the cards numbered at or below it; the overview chain uses the whole plan.
A directory passes when it exists on disk under `worktreeRoot`, or when it is, or is an ancestor of, a created directory.
Otherwise it is one finding, `Check: "verify-package-missing"`, attributed to the card (no card for the overview chain), naming the argument as written; its way forward is to correct the Verify line.
When the argument ends in `_test` and the directory `goPackageDir` gives for it exists or is created, the detail says it is an external test package whose tests live in that directory and names the corrected argument.
A Verify on or after a card with a `Move` group is not checked, and the overview chain is not checked when any card has one, since a Move's destination lives in Intent prose.
Bound: static spellings only, so an argument behind a variable, `bash -c`, an unplaceable `cd`, an import path, a module-wide `./...` (its trimmed directory `.` always exists), `all` and a `.go` file argument pass; a directory an earlier card deletes is not reported, since the card's own verify catches it.
It runs under any `language:`.

`validate` appends it right after `checkVerifyModuleWide`, so `Validate` and `ValidateFormat` both report it and dispatch drops a finding against a completed card as it does for every check; the check-ID list in `validate.go`'s head comment, a file comment outside `validate`'s span that rides on this card, gains `verify-package-missing (checkVerifyPackageMissing, verifymodule.go)` before `pattern-entry-line-cap`.

`TestValidate_VerifyPackageMissing` is table-driven like `TestValidate_VerifyNestedModule`, over a `t.TempDir()` root holding `internal/x/x.go`, `internal/old/o.go` and `nested/pkg/p.go` through `materializeFiles`, collecting the cards of `verify-package-missing` findings, with a `wantDetail` substring column checked against each finding's `Detail`.
A row's plain Verify card is `cardOfType(1, "a", planparser.CardTypeEdit, []string{"internal/x/x.go"})` with its Verify set, never `validCard`, whose `pkg/card<N>.go` target would make `pkg` a created directory.
Firing rows: a card Verify `lyx gate test ./internal/x_test`, whose `wantDetail` is `./internal/x`; overview `go test ./nope` with no card; `go -C nested test ./nope`, resolved against `nested`; `go test ./pkg` from the root, while only `nested/pkg` exists; `go build ./nope`; `lyx gate test -C nested ./nope`; `cd nested && go test ./nope`; a card 1 Verify `go test ./internal/new` with only card 2 creating `internal/new/x.go`; a Create card on `internal/new/x.go#Foo` with Verify `go test ./internal/new_test`, whose `wantDetail` is `./internal/new`; a Delete card on `internal/gone/g.go#`, absent on disk, with Verify `go test ./internal/gone`, since a Delete-only directory is not created.
Silent rows: `go test ./internal/x`; `go -C nested test ./pkg`; `lyx gate test --tags integration ./internal/x -- -run X`; a Create card on `internal/new/x.go#Foo` with Verify `go test ./internal/new`; a Custom card targeting `internal/new/x.go` with Verify `lyx gate test ./internal/new`; a later card's Verify on a directory an earlier card creates; a Rename card whose pair's New side is `plan:internal/new/n.go#New` with Verify `go test ./internal/new`; a Create card on `pkgnew/sub/x.go#Foo`, with `pkgnew` absent on disk, and Verify `go test ./pkgnew`, the ancestor of a created directory; `go test ./...`; `go test example.com/m/x`; `go vet ./internal/nope/x.go`; `cd /abs && go test ./nope`; a Verify `go test ./nope` on a Move card and on a card after it; overview `go test ./nope` in a plan holding a Move card.
`internal/planparser`'s row in `cmd/lyx/testdata/test-budget.yaml` rises from 92 to 93 for the new top-level test.

`TestGateParity_PlanGate` gains a `VerifyPackageMissing` row beside `VerifyModuleWide`, built with `verifyPlanFixture(t, anchorPath, worktreeRoot, "", "go test ./nope")`, wanting a stuck gate and verb and the `verify-package-missing` finding in both, so the plan gate and `lyx loom validate-plan` are pinned to report it alike.

The golden fixture's Verify `go test ./internal/boardcli` and its overview chain resolve against directories `TestValidate_GoldenFixture_ZeroFindings` already materializes, so it stays at zero findings.

Docs in the same commit: `internal/planparser/doc.go`'s shell-chain paragraph names `verify-package-missing` as the third reader of the splitter, and `contracts/specs/loom-plan-spec.md`'s validation-checks list gains it as check 42 after `fabric-vocabulary-name`, stating what it reads, the created-directory rule, the `_test` hint, the Move skip, the bound, that it runs under any `language:` and that both plan gates report it.

**ImpactSummary:** One new check appended to the planparser dispatch list; existing checks, fixtures and the golden plan keep their findings, and a plan with a Verify on a directory that no card creates now blocks at both plan gates.

**Commit:** `2: verify-package-missing`
