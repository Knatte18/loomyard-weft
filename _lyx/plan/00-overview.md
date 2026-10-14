---
format: 5
approved: true
language: go
---

# Plan: plan-gate-gaps

Webster's checks should agree with the code a plan changes, so a correct batch is never rejected and a cross-package break is caught at the card that causes it.
Cards 1 and 2 close the first two gaps with one batching rule (D1): a card that renames or deletes what an earlier card of its candidate batch targets starts a new batch, decided by a new `planparser` ref helper and applied by the cost batchifier.
Cards 3 to 6 close the third (D2 to D4): every card gate ends with one `lyx gate test` step over the root module's `//lyx:guard` tests, found by impactset's marker scan over webster's own walk of the root module, with a prompt line instead of the step when the scan fails or the step is over-long, a refusal when the walk cannot read a path, and an implementer-body rule for fixing a guard failure outside the card's targets.

## Card Index

1 — ref-covers — planparser decides when one ref covers another
2 — rename-batch-boundary — A rename or delete of an earlier card's target starts a new batch
3 — export-guard-scan — Export impactset's guard scan and command limit
4 — guard-stencils — Webster stencils for the guard step
5 — guard-scan — Webster scans the root module for guard tests
6 — guard-step — Every card gate ends with the root module's guard tests

## Shared Decisions

- Every ref-shape test goes through `planparser.RefCovers`, `RefScope` or `RefFile`; neither `internal/batcher` nor `internal/websterengine` tests a ref's shape itself (`PATTERN-ref-shape-registry`).
- The fork-prompt wording for D3 and D4 lives in webster's stencils; Go fills markers only (`PATTERN-stencil-ownership`), and tests pin a marker or a rule's presence.
- A recorded partition in `state.json` is never regrouped; D1 changes only a derivation.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
