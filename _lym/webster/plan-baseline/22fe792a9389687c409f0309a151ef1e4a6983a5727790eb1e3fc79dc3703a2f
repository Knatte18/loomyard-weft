---
format: 5
approved: true
language: go
---

# Plan: Parts-Write and Parts-Plan-Write

Add two opt-in tapestry producers that write the plan in two steps, switched by `tapestry.plan_split`: `Parts-Write` splits the task into parts with contracts in `_lym/parts/parts.md`, and `Parts-Plan-Write` runs one writer seat per part and a chair that joins their cards into one ordinary plan through `lym loom join-parts`.
The cards build bottom-up: config keys, the parts parser and the plan checks' part scope, the join write path and the loomshed checks, the seat engine's writer kind and the producer's writer mapping, the stencils and table builders, the preparations and the registry entries, then the recipe rows and the verbs.
`Plan-Write` keeps its behaviour and stays the default.

## Card Index

1 — tapestry-split-keys — add plan_split and the parts keys to tapestry.yaml with their load-time checks
2 — decision-ids — export the decision record's decision ids from discussionparser
3 — parts-parser — add internal/partsparser, the parts spec and the PATTERN-sole-parsers change
4 — planparser-part-scope — let the pure plan checks read a part with its external exports and shared spots
5 — planglyph-part-scope — let the resolve-backed checks honour a part scope
6 — planparser-join — add JoinParts, the plan format's write path for a joined plan
7 — loomshed-part-checks — add the parts gate, the part check, its gate and Accept, and the join's Go half
8 — writer-seat-directives — ship seat-directive-writer and seat-directive-writer-chair
9 — seat-writer-kind — add the writer seat kind to the seat engine's validation, watch and spec
10 — seat-writer-resume-join — finish writers in resume, the hold deferral and the join
11 — multillm-writer-mapping — map a writer that ended the step onto Stuck or an error naming it
12 — plan-format-block — move the card format and glyph rule into four included plan-format blocks, keeping Plan-Write's prompt line for line
13 — parts-write-table — build the Parts-Write table and ship its stencil and depth fragments
14 — parts-plan-write-table — build the Parts-Plan-Write table and ship the writer and chair stencils
15 — parts-rotators — add the fresh-spawn preparations of the two rows
16 — parts-producers — register the two producers, the parts gate and part_gates, and wire their seams
17 — parts-recipe-rows — add the two rows to the tapestry recipe with names, policy and phase
18 — validate-parts-verbs — add lym loom validate-parts and validate-plan --part
19 — join-parts-verb — add the chair-only lym loom join-parts
20 — settled-guard-writer — name the plan writer the run ran in the settled-verdict refusal

## Shared Decisions

### Decision: reuse-plan-write-decorator

- **Decision:** Both new rows wrap their seat producer in `loomshed.NewPlanWrite`'s commit decorator, each with its own commit seam, rather than a new decorator type.
- **Rationale:** The decorator already commits on every non-empty pointer, Done and Stuck alike, which is what D4 and D6 ask; the shape test tells the rows apart by name and engine.
- **Applies to:** cards 16 and 17

### Decision: part-scope-on-the-plan

- **Decision:** A part check is the ordinary format check over a part directory whose parsed `Plan` carries a `PartScope`, read by planparser's and planglyph's checks; the `planindex.Index` interface does not change.
- **Rationale:** One check set serves the plan gate, the part gate, a writer's Accept and the join, and no index fake changes.
- **Applies to:** cards 4, 5 and 7

### Decision: plan-format-blocks-keep-today-s-order

- **Decision:** The shared card-format and glyph text splits into four blocks, `loom-block-plan-glyphs`, `loom-block-plan-format`, `loom-block-plan-overview` and `loom-block-plan-cards`, cut at the three lines that do not fit every including stencil, and each block is included where its text stands in `loom-template-plan.md` today, so `Plan-Write`'s composed prompt stays the same line for line (the parent's added decision on Plan-Write's prompt); the part writer and the chair include the same four blocks and state their own rule for the three lines.
- **Rationale:** The seat engine includes a block whole at one position and one level deep (`internal/seatengine#validateStencil`), so one block cannot keep today's order and also leave out the lines a part writer must not follow.
- **Applies to:** cards 12 and 14

### Decision: chair-timeout-against-the-effective-plan-timeout

- **Decision:** The chair's default timeout is twice the effective plan timeout, `plan_timeout_min` or shuttle's `run_timeout_min` when it is 0; a set `parts_chair_timeout_min` is refused at config load against a non-zero `plan_timeout_min`, and at table build against `run_timeout_min` when `plan_timeout_min` is 0.
- **Rationale:** tapestry.yaml's load cannot read shuttle's `run_timeout_min`, which the table builder is told.
- **Applies to:** cards 1, 14 and 16

## PATTERN changes

- `PATTERN-sole-parsers` — changes: Parsing plan, parts file, discussion, summary, recipe or halt note, or classifying a plan ref: only its sole parser. (test)
