# Card 2 — card-gate-nested-modules

**Edit:**
- `internal/websterengine#cardGateCommand`
- `internal/websterengine/cardgate_test.go`

**Prosa:**
- `internal/websterengine/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/planparser#NestedModules`
- `internal/planparser#ModuleOf`

**Intent:** Make webster's Go-derived per-card gate satisfiable for a card that targets packages inside a nested module (D7, issue #451's first symptom).

`cardGateCommand` reads `NestedModules(plan, card.Number, worktree)` once, then partitions the surviving package directories (the same Delete-only and no-Go filter as today) with `ModuleOf`.
Root-module directories keep today's `go test -tags integration ./<dir> …` step.
Each nested module adds `go -C <module> test -tags integration <args>`, its arguments module-relative (`.` for the module directory itself, `./<rel>` below it).
The steps join with `&&` in this order: `go build ./... && go test ./...` first and unchanged, the root-module step, one step per nested module in first-seen order of its directories, and `lyx loom lint-comments` last.
Because `NestedModules` counts the card's own and earlier cards' `Create`, `Delete` and `Rename` of a `go.mod`, a module a card creates or moves is tested inside its new module on that card and every later one.

`TestCardGateCommand` gains rows: a directory inside an on-disk nested module beside a root directory; a target that is the nested module's own directory, spelled `.`; a module created by a `go.mod` `Create` target on this card; one created on an earlier card; one created on a later card, which stays a root-module argument; and one moved there by a `Rename` pair.
The rows build the plan with more than one card where the ordering matters.

Docs in the same commit: the card-gate paragraph of `internal/websterengine/doc.go` and the tier 1 bullet of the spec's verify model section say that a nested module's package directories are tested inside that module with `go -C <module>` and module-relative paths.

**ImpactSummary:** one unexported function whose only callers render the card_gates marker; a card with no nested-module directory gets the identical command.

**Commit:** 2: webster's card gate tests a nested module's packages inside that module
