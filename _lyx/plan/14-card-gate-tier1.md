# Card 14 — card-gate-tier1

**Create:**
- `plan:internal/websterengine#gateScope` -> `type gateScope struct`
- `plan:internal/websterengine#gateScopeOf` -> `func gateScopeOf(plan *planparser.Plan, cards []planparser.Card, worktree string) gateScope`
- `plan:internal/websterengine#gateScope.steps` -> `func (s gateScope) steps(tags string) []string`

**Edit:**
- `internal/websterengine#cardGateCommand` -> `func cardGateCommand(plan *planparser.Plan, card planparser.Card, worktree string) []string`
- `internal/websterengine#renderCardGates`
- `internal/websterengine/cardgate_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/planparser#CardTargetDirs`
- `internal/planparser#NestedModules`
- `internal/planparser#ModuleOf`
- `internal/websterengine#holdsGoFile`
- `internal/websterengine#packageArg`
- `internal/websterengine#moduleRelative`

**Intent:**
Drop the card gate to tier 1 of the card's own packages (D1).
`gateScope` holds the surviving package arguments grouped by module, the root module first and each nested module in first-seen order with module-relative arguments; `gateScopeOf` builds it over the cards' target directories with today's filters (a directory whose every target is a Delete, or that neither names Go source nor holds a `.go` file, drops out) against the nested modules as the plan stands once the last of `cards` has run.
`steps(tags)` renders one `lyx gate test` step per module, `lyx gate test [--tags <tags>] <dirs>` for the root and `lyx gate test -C <module> [--tags <tags>] <module-relative dirs>` for a nested module.
`cardGateCommand` now returns the card's gate as its ordered steps: `gateScopeOf` over the one card, `steps("")`, then `lyx loom lint-comments`; a card with no surviving directory runs only the lint.
No `go build ./...`, no module-wide `go test ./...` and no `-tags integration` remain: the batch gate (next card) and the plan verify run the wider checks.
`renderCardGates` renders each card as its pointer bullet with one indented sub-bullet per step, each step its own backticked command, never chained with `&&`, so a fork runs each as its own Bash call.
`cardgate_test.go` keeps its table shape over the step lists: root-only, root plus nested module, a Delete-only directory, a stencil-only card running the lint alone; `beginbatch_test.go`'s rendered-gate assertion follows the new shape.
`internal/websterengine/doc.go`'s card-gate paragraph, which says a card's gate builds and tests everything and runs the integration-tagged tests of its directories, is rewritten to this gate: `lyx gate test` over the card's own packages per module, one step per line, then the comment lint.
`loom-plan-spec.md`'s verify model states tier 1 as the card gate over the card's own packages through `lyx gate test`, and the `caller-uncovered` row's sentences that name the card gate's root build name the batch gate and the plan verify instead.
Bound: a card that breaks a package outside its targets passes its card gate; the batch gate catches it when that package is in the batch, and Webster's gate (the full plan verify) otherwise, before Webster-Review.

**ImpactSummary:** The card gate's shape changes for every fork prompt; its one renderer and two test files follow, and the stencil text around it changes in the next card.
