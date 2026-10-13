---
format: 5
approved: false
language: go
---

# Plan: arbiter-fixes

Three rough edges around the arbiter and the alignment agent.
A re-launch whose arbiter or tackler verdict is already in effect is admitted instead of refused as moved (cards 1–2).
The arbiter no longer races batten's verify retry at a Merge-In or Publish verify halt whose own failure record names a target (card 3).
The arbiter and alignment stencils name the JSON `lyx board get` payload, and the alignment agent settles cross-run landing order from `lyx loom peers`' new `row` and `own_row`, escalating only on a design conflict or an open question (cards 4–5).
The Webster deviation list comes from the glyph scope guard alone, file-granular, and reaches a reader once, in the batch digest (cards 6–8).

## Card Index

1 — relaunch-in-effect-signal — Relaunch skips its goto when Admit reports the target already in effect
2 — shared-relaunch-admit — one admit check lets the arbiter and tackler consume admit a verdict already in effect
3 — arbiter-verify-halt — the arbiter refuses a Merge-In or Publish verify halt whose own record names a target
4 — peers-row — lyx loom peers gives each run's embedded-recipe row and the caller's own
5 — alignment-arbiter-stencils — the alignment agent settles landing order itself; both stencils name the board payload
6 — scope-guard-files — the scope guard names each finding's file and reports uncovered symbol-less files
7 — webster-guard-deviations — record-batch fills the digest's deviations from the scope guard alone
8 — webster-stencils-deviations — the implementer reports no deviations and Master reads the guard's list

## Shared Decisions

- The refusal texts of both consumes stay as they are; card 2 only narrows when they fire.
- `Report.Deviations` stays as an ignored field so `ParseReport`'s known-fields decoding keeps accepting an older fork's key.
- The digest's `deviations` keeps its JSON name and `omitempty`, so the record-batch envelope, the Master's field list and `PATTERN-tolerant-records` are unchanged.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
