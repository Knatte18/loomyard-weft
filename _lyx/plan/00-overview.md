---
format: 5
approved: true
language: go
---

# Plan: review loops converge from round 2, carrying their fresh findings

Review segments stop circling to their budget on fresh edge-case keys.
From round 2 on, the judge converges a segment once its latest round carries no BLOCKING finding and no MEDIUM-or-worse `design` finding on a key already open in an earlier round, and Go carries the open findings of every closing round into the decision record's `## Open risks`, where Plan-Write, rework and Describe each check them.
The template budget drops to three rounds, each review segment can set its own reviewer and fixer models, the fixer can start only after the review is accepted, and `lyx loom resume` resumes a recorded circling decision (issue #446).

## Card Index

1 — carry-over-write — discussionparser's second write path: one segment's carry-over entry in `## Open risks`
2 — bouncer-carry-over — the Bouncer's carry-over seam, called before Approve on every judged Done
3 — carry-over-wiring — `carry_over` recipe key, `Env.CarryOver`, and loomcli's write-and-commit of the decision record
4 — carry-over-consumers — Plan-Write, rework, Describe and the Plan-Review rubric told what an entry means
5 — lighter-convergence-rule — the round-2 rule, the earlier-open facts list, the judge stencil and PATTERN-review-round
6 — review-budget-three — template `review_max_bounces` from 5 to 3
7 — segment-models — six per-segment review and fix model keys in `loom.yaml`
8 — fix-start — `fix_start: parallel | after-review`
9 — resume-circling — `lyx loom resume` over a pending circling decision; resume wording moves to it

## Shared Decisions

### Decision: carry-over-data-owned-by-discussionparser

- **Decision:** the Bouncer hands `discussionparser.WriteCarryOver` structured data (`CarryOver`), and discussionparser alone renders, sanitizes and splices the entry, so the marker grammar, the one-line rendering and the result check live in the record's sole parser.
- **Rationale:** `PATTERN-sole-parsers`; a Bouncer-rendered string would split the bound D2 states across two packages.
- **Applies to:** cards 1, 2, 3

### Decision: segment-name-through-carry-over-key

- **Decision:** a Bouncer row learns its segment name from its own `carry_over` Config key, whose value a loomrecipe test pins equal to the row's `segment`; per-segment models reach a BurlerRound row through `Env.RowReviewModels`, keyed by row name like `Env.SegmentBounces`.
- **Rationale:** a registry `Constructor` sees only the row's name, Config and Env, and `shed-recipe-spec.md` puts every per-row value in Config; neither choice changes the `Constructor` signature or `BouncerConfig.Name`'s log-only role.
- **Applies to:** cards 3, 7

### Decision: pattern-amendment-lands-with-the-rule

- **Decision:** the `PATTERN-review-round` amendment (D7) lands in card 5, the commit that changes the convergence rule (D1), after the carry-over it names (D2, cards 1–4) is in place.
- **Rationale:** the entry stays true at every commit: before card 5 no fix converges on its own, and from card 5 on a fresh-key fix converges and is already carried to the record.
- **Applies to:** cards 1–5

### Decision: halted-paths-keep-start

- **Decision:** only ways forward over a run awaiting at a Bouncer row move to `lyx loom resume`; the Bouncer's Stuck-path refusals and the new carry-over refusal keep `lyx loom start`, as the shed rows of `refusal-spec.md` already do.
  One way forward over an awaiting Bouncer row is a stated exception and keeps `lyx loom start`: `circlingWayMalformed` in `internal/loomcli/circling.go` (refusal-spec row `circling: escalation record malformed`), because its fix needs the round re-escalated, which `start` does, while `resume` with no pending decision refuses.
- **Rationale:** D6 scopes the rewording to the awaiting Bouncer row; a halted run's start already wakes a parked driver or spawns one, and the malformed-escalation fix has no decision for `resume` to act on.
- **Applies to:** cards 2, 9

### Decision: docs-without-doc-go

- **Decision:** `loomengine` and `loomcli` carry no `doc.go`, so their docs are `config.go`'s header, `template.yaml`'s key comments and the verbs' file headers and `Long` texts, which cards 6–9 update; `shedadapters`, `burlerengine` and `discussionparser` update their `doc.go` in the card that changes them.
- **Rationale:** docs land in the card that changes the behavior, in the file that documents it.
- **Applies to:** cards 1, 2, 5, 6, 7, 8, 9

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
