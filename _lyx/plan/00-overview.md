---
format: 5
approved: true
language: go
---

# Plan: run-reentry

A goto, a resume or a stop keeps what the run already settled: a converged or accepted review verdict, the spent review budget and the reason a run is paused.
A review segment re-entered after its verdict settled re-settles an unchanged subject at no review cost, or reviews only the change since then under a small allowance.
A bounce-budget-exhausted halt is lifted only by the parent's resume, never by a goto.
A stop starts no friction session, and a review round whose chair spawned the wrong fork count is retried.
The cards build bottom-up: the state and Shed engine rules first, then the verbs and the driver, the config key and the git reads, the Bouncer's records and re-entry, the recipe and loom wiring, and last the texts and docs.

## Card Index

1 — state-write-file — write raw record bytes atomically through internal/state
2 — shed-budget-episode — a goto keeps the spent budget; a budget halt's resume grants one more; a generation start resets the count
3 — shed-pause-reason — every write of paused records its reason and the requester's label
4 — shed-goto-guard — goto and relaunch consult a told guard before moving a run
5 — shedverbs-pause-label — pause takes a caller label, status shows the pause reason, goto passes the guard
6 — sheddriver-budget-handback — a budget halt goes back to the parent; the paused rule reads the recorded reason; the tackler runs the guard
7 — review-amend-key — review_amend_max_bounces in loom.yaml and darn.yaml
8 — fabric-task-patch — the task patch, its patch-id, merge-in-only history and the weft HEAD
9 — identity-text — the identity text of the decision record and the plan overview
10 — settle-records — the Bouncer's settle and generation records, their liveness and their survival across an archive
11 — bouncer-reentry — the Bouncer records its settle and re-settles an unchanged subject without a review
12 — bouncer-amendment — a changed subject starts an amendment generation under the allowance, scoped to the change
13 — recipe-bouncer-subject — each Bouncer row names its subject seam and takes the allowance
14 — loomshed-rework-records — PR-Rework keeps the review records; the reject's findings are kept for the amendment
15 — loom-subject-wiring — loom and darn tell each review segment its subject, allowance and reject file
16 — loom-goto-guard — refuse a goto that would re-run a settled review, bar the verify retry
17 — budget-way-forward-texts — a budget halt names the resume verb, and no text calls a goto a fresh budget
18 — friction-no-per-stop — a halt writes its friction note and leaves the reflection to Friction-Reflect
19 — burler-forkcount-retry — retry a review round with the wrong fork count, and name how the chair ended
20 — status-spec-docs — the status contract and overview follow the re-entry rules

## Shared Decisions

### Decision: nil seams keep today's paths

- **Decision:** every new Bouncer, Shed and verb seam (`Subject`, `GotoGuard`, `RejectReview`, `BudgetWayForward`) is nil or empty by default and leaves its owner on today's path; only loom's and darn's wiring tells them.
- **Rationale:** batten tells no guard, and every existing test that tells none keeps its meaning, so each card is committable before the wiring card lands.
- **Applies to:** cards 2, 4, 5, 6, 11, 12, 13

### Decision: records beside the run directory

- **Decision:** the settle record, its patch and the generation record live in the Bouncer's run directory, written through `internal/state` with their locks beside them, as PR-Rework's round records are.
- **Rationale:** the run directory is the durable record the wiring commits, and every archive of it carries the three files along.
- **Applies to:** cards 10, 11, 12, 14

## PATTERN changes

- `PATTERN-friction-capture` — changes: Halting a loom run or refusing in webster: Go writes a friction note, reflected by the run's Friction-Reflect row rather than at the stop, and archived only once a reflection covers it.

## verify:

go build ./...
go test -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
