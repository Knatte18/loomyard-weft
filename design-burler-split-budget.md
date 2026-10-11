# Burler: reviewer row then fixer row, lean fixer, per-perch budget and auto-accept

`burler-split-budget` · Task · enhancement, loom, burler · running · Describe

Merged from notes `segment-budget-accept` and `burler-chair-late-spawn` (operator, 2026-10-11).

## Seen (2026-10-11)

Measured over the four newest loom runs (analysis in `.scratch/burler-cost-breakdown.md`, tokencount report `.scratch/token-usage-by-role.md`; both scratch, may be gone):

- Burler plus its Bouncer passes cost about $11.7 per run: reviewer (seat `advisor`) $24.5, fixer (seat `chair`) $18.1, Bouncer $4.2 over the four runs.
- The Burler round runs as one seat table on `internal/seatengine` (since `7882c2450`): the chair (fixer) spawns first, loads its orientation and waits in `lyx shuttle await` for the advisor (reviewer); 57–61% of the fixer's turn time is that wait.
  The wait itself is cheap (its cache stayed warm), but because the fixer spawns first, it is paid for even when the review turns out empty.
- Duplicate orientation is the waste: both seats load the same prompt, explore file (PATTERN overview, rubric, target, fasit), focus file, decision record and plan cards; about 44k of the fixer's ~64k tokens at its wait, $8.36 of its $18.10.
- In 31 archived rounds over six runs the fixer applied every finding and disputed or deferred none.
- Webster-Burler's first rounds yielded 0 BLOCKING, 1 MEDIUM, 1 LOW, 3 NIT and 2 empty reviews; its Opus fixer cost more than its Sonnet reviewer even on empty reviews.
- `loom.yaml` `review_max_bounces` is one budget for every Bouncer review segment (Discussion-Review, Plan-Review, Webster-Review), and a spent budget always escalates to the parent, which answers with `lyx loom circling accept`; nothing accepts on its own.

## Scope

### 1. Reviewer row, then fixer row

The Burler round becomes two Shed rows instead of one seat table, sequenced by the FSM:

1. Burler-Review: a `MultiLLMProducer` with zero advisors whose chair is the reviewer and writes the review file.
2. Burler-Fix: a `MultiLLMProducer` with zero advisors whose chair is the fixer, reads the review file and fixes.

Each review segment's Bouncer `OnStuck` points at Burler-Review, Burler-Review goes on to Burler-Fix, and Burler-Fix returns to the Bouncer (`contracts/recipes/loom-recipe.yaml`, `internal/shedadapters`).
This keeps `MultiLLMProducer` as the one LLM producer, in line with note `seats-replaces-singlellm`.
Rejected: keeping one seat table and having `internal/seatengine` defer the chair's spawn until the advisor delivers; it adds special code to the seat engine.
Rejected (operator): starting the fixer as a fork of the reviewer; Burler had that before, and the fixer started with the reviewer's whole exploration in its context.

### 2. No fixer on an empty review

When the review has no findings, the round skips Burler-Fix and goes straight back to the Bouncer.

### 3. Lean fixer orientation

The fixer gets a constrained fix report, exact instructions on what to change and where, since the reviewer has done the work (operator): the review file and the targets it names, not the reviewer's explore file (rubric, fasit, PATTERN overview).

### 4. Per-perch budget and auto-accept

Each perch, Discussion, Plan and Webster alike, gets its own bounce budget and an auto-accept flag (operator).
With auto-accept on, a spent budget accepts the work as it stands and the run goes on, instead of escalating to the parent.
`review_max_bounces` and `review_circling_checkpoint` interact today (see their comments in the loom config template); the per-perch keys keep that relation clear.
Defaults keep today's behaviour; the hub then sets Webster to one round with auto-accept (operator: for Go code the tests in verify carry the weight).

## Settled model choices (operator, 2026-10-11)

- Webster-Burler runs Sonnet reviewer and Sonnet fixer; hub `loom.yaml` already sets `webster_review` and `webster_fix` to `sonnet[medium]`.
- Discussion and Plan reviewers stay Opus: no Sonnet reviewer there.

## Open

- The fixer can no longer ask the reviewer follow-up questions; check whether the chair uses that today.
- After one Webster round only verify checks the fixer's changes; accepted for code (operator).
