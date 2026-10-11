---
format: 5
approved: false
language: go
---

# Plan: parent-review-wait

Discussion-Write's parent-review wait costs less and keeps its promises.
The gate drops its pass-at-cap mode so every rewrite after a reject is reviewed, and closes the gaps the PR #336 review left: `parent-review` must be last in its gate list, a fresh spawn reuses an empty round, `approve --review` refuses an empty file, `delivered` on a settled round writes nothing, and the review refusals name a status command that can reach the run.
The reviewer brief and the orch role tell both readers never to message the writer.
The wait loop re-reads a pending gate at most every 5 seconds, and an off-boundary expiry probe ends the wait at the bound even when the writer never ends its turn.
While a delivered request waits, the gate pings the idle writer inside its prompt-cache lifetime, at most three times per round, so a late reject resumes it on a warm cache.
D11 needs no card: this run's own round-1 `delivery.json` and `verdict.json` are its proof, which the landing description cites with the gaps the decision record names.

## Card Index

1 — engine-prompt-cache-lifetime — Engine answers a session's warm prompt-cache lifetime; Claude from the role's TTL
2 — keepalive-stencil — the one-line keep-alive stencil, its renderer and the discussion stencil's third message
3 — keepalive-pings — the gate carries up to three counted keep-alive pings per delivered round
4 — keepalive-wiring — loom wiring sets the keep-alive interval from the writer's cache lifetime
5 — parent-review-no-pass-at-cap — drop the gate's pass-at-cap mode, refuse pass_on_cap on parent-review, and set the recipe to two reviewed rounds
6 — parent-review-last-gate — refuse a gate list with an entry after parent-review
7 — prepare-round-reuses-empty — a fresh spawn reuses a latest round that holds no request
8 — approve-review-file-refusal — approve --review refuses a missing, unreadable or empty file
9 — delivered-after-verdict — delivered on a round with a verdict exits 0 and writes nothing
10 — review-refusals-name-status — review refusals name lyx batten status <slug> for a slug, lyx loom status for none
11 — no-reply-to-writer — the brief and the orch role say no message goes to the writer
12 — gate-recheck-throttle — re-evaluate a pending gate with no new arrival at most every 5 seconds
13 — gate-entry-expiry-probe — GateEntry gains an expiry probe the wait loop calls off the boundary, finalizing done
14 — parent-review-expiry-probe — the parent-review entry's read-only probe and its resolver wiring

## Shared Decisions

### Decision: test-budget

Every package this plan touches sits at its `cmd/lyx/testdata/test-budget.yaml` row.
Cards 1, 2, 3, 4, 12 and 13 each raise their package's row by one for the new top-level test they add.
Card 5 deletes five parentreview tests and swaps one shedrecipe test for another, which leaves room for card 14's parentreview test, and every other new case extends an existing test.

### Decision: keep-alive-first

The keep-alive cards run before card 5 because card 3 edits `closures.gate`, whose current body still calls the `closures.logPassedAtCap` card 5 deletes.
Card 5 then removes the pass-at-cap branches from a `closures.gate` that already carries the ping branch.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
