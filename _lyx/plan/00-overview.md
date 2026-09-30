---
format: 5
approved: false
language: go
---

# Plan: Batten run records in the prime leak into every new pair

A pair created by `lyx fabric add` or batten's `Worktree-Create` forks its weft branch from the parent's, and so inherits every committed `_lyx/shed/<slug>/` run directory the parent carries — including prime's batten record for the child's own slug, which makes the child's `Seed-Child` refuse a disagreeing seed.
This plan stops the inheritance at its source: `fabricengine`'s `Add` forks without checking the run-records root out and drops it in the pair's first weft commit, learning the root from a new `shedrun` constructor.
It also makes batten's two prime-lock producers wait, bounded and cancellable, for a contended lock instead of halting `blocked` on the first try.

## Card Index

1 — shedrun-runs-root-rel — export the anchor-relative run-records root from `shedrun`
2 — fabric-add-drops-run-records — `Add` forks without the parent's run records and commits the drop with the origin record
3 — battenshed-prime-lock-wait — `Worktree-Create` and `Worktree-Teardown` wait for a contended prime lock
4 — batten-child-ignores-prime-records — batten integration tests reproducing the reported seed collision
5 — docs-fresh-pair-and-lock-wait — `docs/overview.md` and `CONSTRAINTS.md` state the new behaviour

## Shared Decisions

### Decision: no-checkout fork

The drop keeps the run-records root off the new weft worktree's disk rather than deleting checked-out files, so it deletes nothing and stays outside the Fabric Destruction Chokepoint; card 2 spells out the git sequence.

## verify:

```bash
go build ./...
go test ./...
go test -tags integration ./internal/shedrun/... ./internal/fabricengine/... ./internal/battenshed/... ./internal/battencli/...
```
