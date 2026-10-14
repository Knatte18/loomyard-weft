---
format: 5
approved: false
language: go
---

# Plan: overview-seats

Three changes to the seat and producer layer and to how a run shows in its overview window.
The loom checklist pane becomes one combined frame: recipe rows in order with their visit lines under them, review segments grouped by round in run order, arbiter attempts in the round they decided, a bounce read as `bounced` from a `routed_to` Shed now records, and paused time left out (cards 1 to 7).
Seat tables name their seats: the first seat is the unnamed lead whose role is the prefix alone, later seats carry recipe or config names (card 8).
`SingleLLMProducer` goes: every LLM writer row runs on `MultiLLMProducer` over a seat table, with the prompt amendment the single producer had (cards 9 to 18), and Discussion's and Plan's `*_producer` switches become named advisors and lens-fork keys, migrated from old files (card 19).
The decision record's scope names a `claimed-scope-edit` row joining the converted rows if one lands first; none exists in the tree this plan is written against, so the converted rows are Discussion-Write, Plan-Write, Darn-Write, Describe and PR-Rework.
A symbol a card stops calling is deleted by a later card once no later card edits code that names it (cards 7, 18 and 20).

## Card Index

1 — shed-bounce-routed-to — Shed records the OnStuck target on the Stuck entry it routes, and folds only matching targets
2 — loomrecipe-rows — expose each recipe row's name, segment, OnStuck and Bouncer flag
3 — output-round — map a round file path to its round number in shedadapters
4 — checklist-combined-frame — one frame of recipe rows with their visit lines; bounced, stuck and escalated words
5 — checklist-review-rounds — render a review segment as one block grouped by round in run order
6 — checklist-arbiter-lines — show each arbiter attempt after the entry its stop key names
7 — checklist-pause-net-time — subtract the pause record's coverage from every visit's time
8 — seat-lead-naming — the first seat is the unnamed lead; later seats are named
9 — seat-lead-amendment — a table's fresh-start amendment appended to the lead's prompt only
10 — multillm-amendment — MultiLLMProducer's preparation returns an amendment; not-started errors carry the start error
11 — discussion-write-seats — Discussion-Write always runs on seats; the DiscussionSeats row goes
12 — plan-write-seats — Plan-Write runs on a one-seat table built by PlanTable
13 — pr-rework-seats — PR-Rework runs on a one-seat table built by ReworkTable
14 — describe-seats — both Describe rows run on one-seat tables built by landingshed
15 — darn-write-seats — Darn-Write runs on a one-seat table built by DarnTable
16 — shuttle-seam-move — rename singlellm.go to shuttle.go and move a shedrecipe test helper out of the file card 17 deletes
17 — single-llm-delete — delete SingleLLMProducer, SpecSource and the SingleLLM row; reword every comment citing them
18 — retired-builders-delete — delete the spec builders no row calls any more: DiscussionSpec, PlanSpec, ReworkSpec, DescribeSpec, DarnDescribeSpec and their composers
19 — loom-config-axes — retire the producer keys; named discussion_advisors, discussion_lenses and plan_lenses; MigrateConfig rewrites old files
20 — seat-names-delete — delete RoleChair, RoleAdvisor and AdvisorName, which no code references any more

## Shared Decisions

### Decision: lead-by-empty-name

- **Decision:** the seat engine identifies the lead as the seat with an empty `Name`, which `Table.Validate` pins to the first seat, and `SeatRole(prefix, "")` is the prefix alone.
- **Rationale:** a `Seat` holds a function field and cannot be compared, and an index threaded through every helper is noisier than a name every later seat must carry anyway.
- **Applies to:** cards 8 to 20

### Decision: rows keep the old producer meaning until card 19

- **Decision:** cards 11 and 12 build Discussion's advisors and both rows' lens forks from `discussion_producer` and `plan_producer` as today; card 19 switches the builders to the new keys in the same commit that retires the old ones.
- **Rationale:** every card builds and passes on its own, and no intermediate commit starts the hub's dormant advisor.
- **Applies to:** cards 11, 12 and 19

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
