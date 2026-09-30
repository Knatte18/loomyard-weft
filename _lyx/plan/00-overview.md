---
format: 5
approved: true
language: go
---

# Plan: status strand by driver

A loom run's tmux session shows one driving surface, chosen by the run's recorded driver: a go-driven run shows the `loom-status` strand, and an llm-driven run shows the ly-drive strand and never the status strand.
Today `start` and `step` both call `ensureStatusStrand`, so every llm run carries the band and every ly-drive step re-adds it.
This plan makes `step` bring reed up with no strand work, and makes `start` branch on the driver it already reads: go keeps today's keep/replace/add, llm removes every same-named status strand, non-recursively, warning on failure.
`lyx loom status` and pane-height config are untouched.

## Card Index

1 — status-strand-removal-helper — add `removeStatusStrands`, which removes every `loom-status` strand non-recursively and warns on failure, with unit tests
2 — step-drops-status-strand — `loomPreStep` calls reed `Up` in place of `ensureStatusStrand`; smoke tests pin that step adds none and leaves an existing one alone
3 — start-strand-by-driver — `start` runs reed `Up`, then keeps the status strand on go or removes it on llm; invariant, help text, overview and smoke tests follow

## Shared Decisions

- The driver read stays single-site: `start` branches on the value `seedAndCommitBootstrap` returns, and `step` keeps discarding it.
- Reed `Up` runs on both of `start`'s arms before any strand work, and in `step` in place of `ensureStatusStrand`; a failed `Up` fails either verb exactly as a failed `ensureStatusStrand` does today.
- The llm arm's removal never fails `start`: a leftover band costs screen rows, not the run.
- Live-substrate behaviour (reed on a concrete `*reedengine.Engine`) is pinned in the `smoke` tier; the removal logic itself is pinned offline through `removeStatusStrands`'s function-value seams.

## verify:

```bash
go build ./...
go test ./internal/loomcli/... ./internal/shedcli/...
go test -tags integration ./internal/loomcli/... ./internal/shedcli/...
go vet -tags smoke ./internal/loomcli/... ./internal/shedcli/...
```
