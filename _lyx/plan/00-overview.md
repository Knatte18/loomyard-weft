---
format: 5
approved: true
language: go
---

# Plan: run-parentage

A run in a task pair learns its parent from the worktree the pair was created from, recorded in fabric's origin record and resolved at use to that worktree's orch name, instead of from the caller's `LYX_STRAND_NAME` (#341).
The parent-review gate reviews every rewrite after a reject up to a cap of three rejected rounds, escalates at the cap by halting the run `blocked`, and holds its delivery prompt while the reviewer has no live session.
Batten's Run-Shed waits through a `paused`, `blocked` or `failed` child instead of exiting `failed` (#338).

## Card Index

1 — origin-parent-worktree — record the creating worktree as `parent_worktree` in fabric's origin record
2 — parent-resolver — resolve a pair's parent agent name in `hubgeom` and feed reed's `LYX_PARENT` from it
3 — gate-hold-terminal — let a must-pass gate entry hold the run and fail terminally, with the reason on the outcome
4 — review-store-rounds — round store: reject count, request cap, fresh-spawn preparation, cap-reject supersede
5 — review-gate-rounds — parent-review gate loops over rounds and fails terminally at the reject cap
6 — review-gate-hold — parent-review gate holds its prompt while the reviewer is not live
7 — parent-review-recipe — Discussion-Write's parent-review entry becomes must-pass with `attempts: 3` as its cap
8 — loom-reviewer-resolved — loom wires the reviewer through the resolver and the liveness seam through reed
9 — seed-parent-retired — stop writing the seed's parent; parentage rule in CONSTRAINTS.md
10 — review-approve-supersede — `lyx loom review approve` supersedes the cap's reject on a halted run
11 — batten-halted-wait — Run-Shed waits budget-exempt through a halted child

## Shared Decisions

### Decision: one parent resolver

- **Decision:** `hubgeom.ResolveParent` (card 2) is the only code that turns a pair into a parent agent name; reed geometry and loom's parent-review wiring both call it, and neither `parentreview` nor `reedengine` reads an origin record or a seed for it.
- **Rationale:** both consumers are told-geometry engines, and one resolver keeps the legacy-seed fallback in one place.
- **Applies to:** cards 2, 8, 9

### Decision: the store decides the cap

- **Decision:** the reject cap is counted from the round store's rejected rounds, never from the shuttle engine's in-memory failure count; the cap's reject returns `GateResult.Terminal`, which finalizes without a re-prompt.
- **Rationale:** the in-memory count resets on attach, so only the store survives an attach, a driver restart or a resume.
- **Applies to:** cards 3, 4, 5, 7, 10

## verify:

go build ./...
go test -tags integration ./...
go vet -tags smoke ./...
