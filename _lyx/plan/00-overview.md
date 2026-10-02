---
format: 5
approved: false
language: go
---

# Plan: Parent review as a step in Discussion-Write's gate

Port millhouse's `orch-review`/`orch-wait` into Discussion-Write's own producer gate.
Once the mechanical `discussion` gate passes, a `parent-review` gate entry opens a review request for the run's parent (the seed's `Parent`), has the live writer notify it through `SendMessage`, and waits on a verdict the parent submits through new `lyx loom review` verbs.
A reject re-prompts the still-live writer once; the next arrival re-runs `discussion` and is let through to Discussion-Review.
The wait is a new pending gate state the shuttle wait loop holds without stalling its liveness and deadline checks, and status in both the task worktree and the prime shows it.

## Card Index

1 — gate-pending-state — `shuttleengine` gate results gain a pending state the wait loop holds at turn boundaries
2 — loom-parent-review-config — `parent_review_wait_min` and the parent-review directory accessors in `loomengine`
3 — parent-review-stencils — delivery-prompt and reviewer-brief stencils plus their compose functions
4 — parent-review-store — `internal/parentreview`'s on-disk round store and verb-facing transitions
5 — parent-review-gate — the gate and finalize closures over the store
6 — recipe-parent-review-gate — `parent-review` joins `shedrecipe`'s gate vocabulary, pass-on-cap only
7 — loom-review-verbs — `lyx loom review notify|delivered|approve|reject` with refusal rows
8 — shedverbs-waiting-hook — a told `Waiting` hook on the generic status verb
9 — loom-status-review-wait — `lyx loom status` reports the reviewer wait
10 — batten-review-wait — batten's InnerRun wait message names the child's reviewer wait
11 — discussion-stencil-parent-review — Discussion-Write reads the board and handles parent-review prompts
12 — wire-parent-review — enable the entry on Discussion-Write and commit the round directories with the discussion
13 — orch-review-fork — the hub handles a review notice through a one-shot fork
14 — parent-review-integration — integration test and live smoke for the whole exchange

## Shared Decisions

- **Gate contract additions (card 1).** `GateResult` gains `Pending`, `Send` and `SendFailedWayForward`; `GateEntry` gains an optional `Final` closure `finalize` evaluates instead of `Gate`; `GateEntryState` gains `waiting`.
  Only a `PassOnCap` entry may return pending, enforced at run time by shuttle and at construction time by `shedrecipe`.
- **Turn boundary.** The writer is at a turn boundary when its last arrival was Done and the loop has sent it nothing since; every gate send and every pending re-evaluation happens only there.
- **On-disk layout (cards 2 and 4).** Rounds live at `_lyx/reviews/parent-review/round-<N>/` with `request.json`, `brief.md`, `delivery.json`, `verdict.json` and `review.md`; the store's lock files live under the `.lyx` mirror of that directory.
  `loomengine` declares the segment; `internal/parentreview` receives both directories told.
- **Provider wording.** The SendMessage instruction lives only in the `loom-template-parent-review-delivery` stencil, rendered by `loomengine` and handed to the closure through a seam; `shuttleengine`, `reedengine` and `internal/parentreview` carry no provider words.
- **Round lifecycle.** A fresh Discussion-Write spawn calls `Store.BeginRound`; an attach continues the latest round; the closure opens a request in the latest round when it has none.
  Verbs, closure and status act on the latest round only.

## verify:

```bash
go build ./...
go test ./...
go test -tags integration ./...
go vet -tags smoke ./...
```
