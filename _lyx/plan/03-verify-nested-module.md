# Card 3 — verify-nested-module

**Create:**
- `plan:internal/planparser#checkVerifyNestedModule` -> `func checkVerifyNestedModule(plan *Plan, worktreeRoot string) []ValidationError`
- `plan:internal/planparser#crossModuleCommand` -> `type crossModuleCommand struct`
- `plan:internal/planparser#crossModuleCommands` -> `func crossModuleCommands(chain string, modules []string) []crossModuleCommand`
- `internal/planparser/verifymodule.go`

**Edit:**
- `internal/planparser#validate`
- `internal/planparser/validate_test.go`
- `internal/loomcli/parity_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `plan:internal/planparser#NestedModules`
- `plan:internal/planparser#ModuleOf`

**Intent:** Add the blocking plan check `verify-nested-module` (D8), so a plan whose author-written gate commands cannot pass by construction is refused at the plan gate instead of at run time.

`crossModuleCommands` scans one shell chain and returns each `go` command whose relative package argument resolves into a nested module other than the module the command runs in; `crossModuleCommand` holds the command text and that module.
The chain splits into segments at `&&` and `;`, and each segment into whitespace-separated fields.
The run directory starts at the worktree root; a segment that is a literal `cd <dir>` moves it, relative to the current run directory.
A `cd` to an absolute path or out of the root ends the scan of that chain, since the check only refuses what it can place.
In a `go` segment, `-C <dir>` (or `-C=<dir>`) before the subcommand sets that command's directory, resolved against the current run directory.
After the subcommand, a field opening with `-` is a flag: spelled with `=` it is whole, and spelled without, it consumes the next field when go's flag of that name takes a value (`-o`, `-run`, `-tags`, `-count`, `-timeout` and the rest of the `go build`/`go test` value flags), so `-o ./bin` and `-run .` are never package arguments.
A remaining field that is `.`, `..` or opens with `./` or `../` is a package argument; its directory, with a trailing `/...` dropped, joins the command's directory, and `ModuleOf` names its module.
The command is reported when that module is nested (not `.`) and differs from `ModuleOf` of the command's directory.
So `./...` from the root, which the go tool limits to the root module, is silent.

`checkVerifyNestedModule` runs `crossModuleCommands` over each card's `Verify` value with `NestedModules(plan, card.Number, worktreeRoot)`, and over the overview's `Verify` chain with the whole-plan set, `NestedModules` through the plan's highest card number.
The overview's lines are already chained with ` && `, which is how they run, so a `cd` on one line carries into the next exactly as it does at run time.
One finding per reported command: `Card` is the card's ID, or empty for the overview; the detail names the card or "the overview's `## verify:` section", the command and the module, and its way forward is to run the command as `go -C <module>` with module-relative package paths.
It runs under any `language:`, since it reads `go` commands rather than glyphs.
`validate` calls it after `checkCommitSubjectMismatch`, so `ValidateFormat`, `Validate`, both plan gates, `lyx loom validate-plan` and dispatch all report it through the one function (`PATTERN-gate-self-check-parity`).

Bound, stated in the spec row: the check only refuses and removes no guard; it misses a command built from a variable, inside `bash -c`, after a subshell, or with an import-path package argument, and webster's gates still catch those at run time.
After a `Move` of a `go.mod`, which `NestedModules` does not count, its verdict on a later card can differ between the plan gate and dispatch.

Docs in the same commit: a new spec row for `verify-nested-module` inserted after `commit-subject-mismatch`, matching `validate`'s order, with the later rows renumbered; the ID joins the check list in `validate.go`'s file comment.

Tests: `TestValidate_VerifyNestedModule` in `validate_test.go`, one table over a `t.TempDir()` worktree holding `nested/go.mod`, with rows for a card `Verify:` testing `./nested/pkg` from the root (fires), an overview line doing the same (fires, empty `Card`), `go -C nested test ./pkg` (silent), `cd nested && go test ./pkg` (silent), `cd nested && go -C ../other test ./pkg` resolving against the `cd` (fires when `other` is another nested module), `go build -o ./nested/bin ./cmd/x` and `go test -run . ./internal/x` (silent), `go test ./...` from the root (silent), a module created by a `go.mod` `Create` target on a card after the `Verify:` card (silent) and on the same card (fires), and `language: none` (still fires).
The new top-level test raises `internal/planparser`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.
`TestGateParity_PlanGate` in `internal/loomcli/parity_test.go` gains a row beside `FabricReference`: a worktree holding `nested/go.mod` and a `## verify:` line `go test ./nested/...`, which the gate and `validate-plan` both answer stuck with `verify-nested-module` in their findings.

**ImpactSummary:** one new call in `validate`'s fixed list; every plan gate, `validate-plan` and dispatch gain the blocking check, and a plan without nested-module commands is unaffected.

**Commit:** 3: plan check verify-nested-module refuses a gate command that tests a nested module from outside it
