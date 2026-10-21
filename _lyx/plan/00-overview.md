---
format: 5
approved: false
language: go
---

# Plan: scratch-defeats-test-cache

In a fabric worktree, files an agent creates, changes or removes under `.scratch/` change the modification time of the module root's real `.scratch` entry, which every source-scan test that opens the module root hashes into its `go test` cache key.
Fabric wiring makes the warp root's `.scratch` a directory link into the pair's weft-side `.lyx`, whose own `Lstat` stays fixed, with an exclude pattern that matches a link, convergence through `lyx fabric reconcile`, an unlink in `lyx fabric remove`, and cleanup working through the link.

## Card Index

1 — scratch-link-wiring — the scratch step: link, migration, exclude and `path_renamed` under every wiring verb, with unwire and remove leaving the link to it
2 — reconcile-converges-scratch — reconcile runs the step on every managed pair and reports it; the stale sweep never claims the link
3 — remove-unlinks-scratch — `lyx fabric remove` unlinks the root `.scratch` link through the gate before removing the warp worktree
4 — cleanup-follows-scratch-link — cleanup's scratch segment documented and pinned working through the link; the Windows check joins `fabric-windows`

## Shared Decisions

- The scratch step lives in `internal/fabricengine/scratchlink.go`; its tests live in a new `scratchlink_integration_test.go` under the `integration` tag, which every card's new test joins.
- A step outcome is two strings, what it changed and what it left with its way forward; reconcile appends both to the pair's Detail, and the wiring verbs log the left one through `logger.Warn`, since their results carry no detail field.
- A created or restored target is recorded as `dir_created`, so the mutation oracle's omission direction covers the new weft-side directory outside a freshly created weft worktree.
- The scratch link's skip in `Unwire`, `Remove` and `applyStaleRemoval` drops the one name `.scratch` at a root anchor only, where `scanOnDiskJunctionNames` sees the root link.

## verify:

go build ./... && lyx gate test --tags integration ./internal/fabricengine/... ./internal/cleanupcli/... ./cmd/lyx
