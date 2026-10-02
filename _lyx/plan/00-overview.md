---
format: 5
approved: false
language: go
---

# Plan: a PR-Rework round starts a new plan generation

A PR-Rework round stops appending cards to the existing plan and starts a new plan generation instead.
Go archives the live generation (its plan, Webster's run record, and the Plan-Review and Webster-Review run directories) into the round's own directory before the rework session runs, and the session writes a whole new plan numbered on from the retired generation through a new `first_card` overview key.
The rework gate checks the whole new plan, the round records whether the generation is exempt from Plan-Review, and the recipe routes every generation through Plan-Review's own segment, which skips its judge only for an exempt generation.
Webster starts a fresh run per generation, Webster-Review reviews only the live generation's commits, and Describe sees every generation's run record.
Two review-stencil fixes ride along: the discussion and plan rubrics ask the attack-surface question, and a Burler focus directive becomes self-contained.

## Card Index

1 — plan-first-card — parse and check an optional first_card overview key so card numbers continue across generations
2 — review-exempt-classifier — classify a plan as exempt from Plan-Review only when every card is Prosa on a non-source file
3 — webster-archive-run-record — move webster's durable run record into a told directory so the next run starts fresh
4 — rework-told-values — let the PR-Rework producer decide what its session is told, via a session factory
5 — rework-archive-generation — archive the live generation into the round and have the session write a whole new plan
6 — retire-plan-fingerprint-rebaseline — remove the rebaseline PR-Rework no longer calls and pin the fresh-run behaviour
7 — rework-gate-whole-plan — the rework gate checks the whole new plan plus the told first_card
8 — bouncer-skip-seam — give the Bouncer an optional skip seam that approves without a judge
9 — plan-review-skip-predicate — decide from committed state whether the live generation skips Plan-Review
10 — rework-routes-through-plan-review — route PR-Rework to Plan-Bouncer and wire the skip seam
11 — describe-every-generation — tell Describe every generation's run record, oldest first
12 — plan-review-rework-answer-key — add the live generation's findings to Plan-Review's answer key
13 — webster-review-rework-range — review only the live generation's own commits in a rework generation
14 — attack-surface-question — ask what a new edge or downgraded guard can skip or let through, in rubrics and writer stencils
15 — self-contained-focus — a focus directive restates its sites and never sends the reviewer to prior-round files

## Shared Decisions

### Decision: generation-round-rule

The live generation's round is the highest committed rework round whose `record.json` `first_card` equals the live overview's `first_card`; with none, the live plan is generation 0.
Go (card 9's predicate) and the review agents (cards 12 and 13, through their rubrics) apply this one rule, and nothing else marks a generation.

### Decision: committed-round-carries-a-class

A rework round counts as committed only when its `record.json` at HEAD carries the class.
The archive step writes `record.json` without a class as its completion marker, and loom's status commit sweeps `_lyx/loom/`, so a classless record can reach HEAD before the session runs; counting it would skip the session.

### Decision: archive-layout-owned-by-loomshed

`internal/loomshed` declares the round layout once: `round-<N>/` holds `findings.md`, `record.json`, `coverage.md` and `prior-generation/`, which holds `plan/`, `webster/` and `reviews/<run_subdir>/`.
Every path is built from the existing accessors (`planparser.PlanDir`, `loomengine.LoomReworkDir`, `loomengine.LoomReviewsDir`, `websterengine.Dir`), never an `_lyx` literal, and webster receives its archive destination as a told path.

## verify:

go build ./...
go vet ./...
go test ./...
go test -tags integration ./internal/planparser/... ./internal/websterengine/... ./internal/loomshed/... ./internal/shedadapters/... ./internal/shedrecipe/... ./internal/shedbuild/... ./internal/loomengine/... ./internal/loomcli/... ./internal/loomrecipe/... ./internal/landingshed/... ./contracts/...
go vet -tags smoke ./internal/planparser/... ./internal/websterengine/... ./internal/loomshed/... ./internal/shedadapters/... ./internal/shedrecipe/... ./internal/shedbuild/... ./internal/loomengine/... ./internal/loomcli/... ./internal/loomrecipe/... ./internal/landingshed/... ./contracts/...
