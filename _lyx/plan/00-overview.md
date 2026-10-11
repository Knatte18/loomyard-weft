---
format: 5
approved: false
language: go
---

# Plan: small-fixes-batch

Land the six small, independent fixes the decision record names (issue #531): the darn writer's prompt reads the board entry with the JSON `board get` form and points at a final-summary spec that is deployed beside the plan spec; `docs/overview.md` states the recipes batten accepts; batten's terminal window opens once per batten run; `lyx config reconcile` raises a below-floor `shuttle`, `orch` or `loom` value through a `Migrate` hook and reports it; `lyx fabric unwire`, `lyx board prune` and `lyx board retire-legacy` change nothing without `--apply`; the `board` group's index note prints under the operator audience only; and `WriteCarryOver` detects a concurrent writer of the decision record before and after its rename.
Each card is one fix and commits on its own; no card depends on a later one.

## Card Index

1 — darn-board-get-json — the darn and landing describe stencils read the board entry with the JSON `board get` form
2 — final-summary-spec-rename — rename, embed and register the final-summary spec so it deploys under the specs dir's `loom/` family, and move every reference
3 — overview-batten-recipes — the batten paragraph names `loom`, `darn` or empty as the accepted recipes
4 — batten-terminal-marker-reset — the terminal-opened marker is cleared where batten seeds a fresh status, never on a child re-spawn
5 — reconcile-floor-migrate — `Migrate` hooks for shuttle, orch and loom raise a below-floor key to its floor and report the rewrite
6 — fabric-unwire-apply — `lyx fabric unwire` is a read-only dry run without `--apply`
7 — board-prune-retire-apply — `lyx board prune` and `lyx board retire-legacy` are read-only dry runs without `--apply`
8 — index-note-audience — an index note prints only under the audience its annotation names; the board group names operator
9 — carry-over-concurrent-write — `WriteCarryOver` refuses before its rename and errors after it when the record changed under it

## Shared Decisions

### Decision: apply-flag-idiom

- **Decision:** the three destructive verbs take `--apply`, the name `lyx fabric prune` and `lyx fabric cleanup` already use; a bare run is a read-only dry run that prints the applied run's fields filled with what it would change plus `"applied": false`.
- **Rationale:** decision D6 and assumption A4; one idiom across fabric and board.
- **Applies to:** cards 6, 7

### Decision: test-budget-raises

- **Decision:** a card that adds a new top-level test raises that package's row in `cmd/lyx/testdata/test-budget.yaml` by one in the same commit; every other card extends an existing test or table.
- **Rationale:** every targeted package sits at its budget today, so a new test fails `cmd/lyx/testbudget_test.go` without the raise.
- **Applies to:** cards 5, 6, 9

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
