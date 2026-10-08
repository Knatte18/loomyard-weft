---
format: 5
approved: false
language: go
---

# Plan: plan gate checks without false findings or unsatisfiable gates

`caller-uncovered` stops matching a re-signed or deleted member by name alone: it resolves a reference by import path, and then by type through `go/packages`, falling back to the import-path scan wherever type information is missing.
A gate command that tests a nested module's packages runs inside that module, both in webster's Go-derived card gate and in the commands a plan author writes, and the new blocking check `verify-nested-module` refuses an author-written one that cannot pass.
The Plan-Review rubric binds a proposed fix to the gate's checks, the plan template states the `go -C` form, and `CLAUDE.md` tells a card to list the test budget it raises.
Sources: board issues #461, #451 and #460, and the `caller-scan-import-path` note.

## Card Index

1 — nested-modules — planparser derives a plan's nested modules through a card, and the module a directory belongs to
2 — card-gate-nested-modules — webster's card gate tests a nested module's packages with `go -C <module>`
3 — verify-nested-module — new blocking plan check over card `Verify:` values and the overview's `## verify:`
4 — caller-scan-import-path — caller-uncovered's scan resolves by import path, collapses name-only method matches, and reaches nested modules
5 — caller-scan-types — caller-uncovered resolves references with go/types through a bounded, logged, offline load
6 — plan-review-fix-rule — Plan-Review proposes only admissible fixes; the plan template states the `go -C` form
7 — test-budget-rule — `CLAUDE.md` line on listing `cmd/lyx/testdata/test-budget.yaml`

## Shared Decisions

### Decision: one nested-module derivation

`planparser.NestedModules` and `ModuleOf` (card 1) are the only code that decides what a nested module is.
The card gate and `verify-nested-module` read them through a card or the whole plan; the caller scan reads the on-disk set, `through` 0, since it judges the tree as it stands.

### Decision: the type loader is a seam, and the untagged tier never spawns

`planGatePass` takes a `typesLoader`; the exported entry points pass `defaultTypesLoader`, the real `goListLoader`.
`internal/planglyph`'s `TestMain` replaces `defaultTypesLoader` with a loader that fails without spawning, so every untagged test runs the fallback, and the integration test hands `planGatePass` a `goListLoader` directly.
The load timeout is the loader's field, never a package-level duration.

### Decision: module paths are read with golang.org/x/mod

`importPathOf` reads a `go.mod`'s module path with `golang.org/x/mod/modfile.ModulePath`, which card 4 makes a direct requirement; `golang.org/x/tools`, which card 5 adds, requires it anyway.

### Decision: the spec's rows are cited by check ID

`verify-nested-module` takes the spec row after `commit-subject-mismatch`, matching `validate`'s order, and the later rows renumber; every card names a row by its check ID, never by its number.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
