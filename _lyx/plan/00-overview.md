---
format: 5
approved: false
language: go
---

# Plan: pair-lifecycle

Removing a pair ends its reed session first, finishes a half-removed pair, preserves every run record in the archive tag, and leaves no landed branch behind locally or, with `--remote`, on origin.
Starting a run never adopts a driver strand that is about to remove itself.
The fabric engine changes land first (refusal probe and record commit, finish path, remote task-branch deletion, remote sweep), then reed's two additions and loom start's retiring handling.
Then come the `internal/pairteardown` composite and its two callers, the deletion of the helpers it makes dead, and the invariant with its docs.

## Card Index

1 — remove-refusal-probe-and-record-commit — expose Remove's refusals as a read-only probe and commit pending sibling records before the archive
2 — remove-finishes-half-removed-pair — Remove finishes a half-removed pair, reports its steps and a stray path, and returns a typed not-found
3 — remove-remote-deletes-landed-task-branch — remove --remote deletes the landed task branch on origin, leased, through a new remote-tip gate
4 — cleanup-remote-task-branch-sweep — engine method that classifies and, with apply, deletes leftover task branches on origin
5 — cleanup-remote-cli-with-open-prs — cleanup --remote fetches open PR heads and composes the task-branch sweep into its envelope
6 — reed-end-session-by-name — engine-less session end for a gone worktree, tidying the server after the last session
7 — reed-retiring-mark — reed remove --detach marks the strand retiring; the detached remover no-ops on a gone guid
8 — loom-start-replaces-retiring-driver — loom start removes a retiring driver strand and spawns a fresh driver
9 — pairteardown-composite — quiet wait, refusal probe, session end, then removal, in one new package
10 — fabric-remove-through-composite — lyx fabric remove calls the composite; envelope and help updated
11 — batten-teardown-through-composite — batten's teardown closures call the composite's two phases; not-found is done
12 — delete-half-removed-helpers — delete RemovePairBranch, PairBranchResult and PairSiblingRemnant
13 — pair-teardown-invariant-and-docs — new CONSTRAINTS invariant with its tripwire, module doc, overview and package docs

## Shared Decisions

### Decision: composite-in-its-own-package

- **Decision:** the teardown sequence lives in `internal/pairteardown`, which imports `fabricengine`, `reedengine`, `hubgeom`, `loomengine` and `shedrun`.
  `fabricengine` imports none of `reedengine`, `loomengine` or `githubclient`.
- **Rationale:** `reedengine` imports `fabricengine`, so fabric cannot end a session itself, and two call sites must not drift.
- **Applies to:** cards 9, 10, 11, 13

### Decision: two-phase-api

- **Decision:** the composite exposes `EndSession` (quiet wait, refusal probe, session end) and `RemovePair`, plus `Run` that sequences both.
  `fabric remove` calls `Run`.
  Batten keeps its two-closure `TeardownDeps` and calls the two phases, so its producer still reports which half failed and still surfaces the abandoned session.
- **Rationale:** batten's producer already guarantees shutdown-before-removal and distinguishes the halves in its stuck reasons; collapsing it to one closure would lose that.
- **Applies to:** cards 9, 10, 11

### Decision: caller-policy-on-busy-driver

- **Decision:** `fabric remove` passes `RemoveQuietWait` (2 minutes, a constant) with `RefuseWhenBusy`.
  Batten passes a zero wait without it, since it already waited its `driver_exit_grace_s`.
- **Rationale:** the wait never kills a working driver on the operator path, and an unbounded wait would hang the hub's land step.
- **Applies to:** cards 9, 10, 11

### Decision: vocabulary-neutral-exports

- **Decision:** every new fabric export a non-owner calls is named without a side's name: `Topology.RemoveRefusal` and `ErrPairNotFound`.
  `internal/pairteardown` passes `RemoveResult` through whole instead of naming its side-named fields.
- **Rationale:** the Fabric Vocabulary Invariant's scan matches the token inside identifiers.
- **Applies to:** cards 1, 2, 9, 11

### Decision: remote-sweep-composed-in-cli

- **Decision:** `Topology.Cleanup` keeps its signature; the origin task-branch sweep is the separate `Topology.CleanupRemoteWarp`.
  `fabriccli` calls it after `Cleanup` with the open-PR set it fetched, and skips it with a reason when GitHub is unreachable.
- **Rationale:** `fabricengine` stays free of GitHub calls, the existing callers and tests of `Cleanup` stay untouched, and an unreachable GitHub fails toward keeping branches.
- **Applies to:** cards 4, 5

### Decision: closed-board-item

- **Decision:** the `main-weft` item needs no code change; card 13 adds its one doc sentence.
  Closing the board item is the hub's job at landing, not a card.
- **Applies to:** card 13

## verify:

go build ./...
go test ./...
go test -tags integration ./internal/fabricengine/... ./internal/fabriccli/... ./internal/reedengine/... ./internal/reedcli/... ./internal/loomcli/... ./internal/pairteardown/... ./internal/battencli/... ./internal/battenshed/... ./cmd/lyx/...
go vet -tags smoke ./internal/reedengine/... ./internal/reedcli/... ./internal/loomcli/...
