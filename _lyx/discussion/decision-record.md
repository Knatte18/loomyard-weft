# Decision record — plan check projects lint-pattern per card

## Goal

The plan gate's PATTERN headroom check projects the overview card by card, the way each card's `lyx loom lint-pattern` step measures it, so a plan whose intermediate card grows the overview past the headroom line fails at plan review rather than at a fork's card gate.

## Scope

- In: `internal/loomshed/patternplan.go` gains a per-card headroom check over the pending cards in plan order, beside the existing whole-plan `pattern-headroom` check.
- In: the docs that state the plan gate's PATTERN checks: `internal/loomshed/doc.go` and `contracts/specs/loom-plan-spec.md`.
- In: table rows in `internal/loomshed/gates_test.go` covering the per-card projection.
- Out: renumbering helpers that rewrite prose card references; the renumber verbs keep reporting prose mentions as today.
- Out: any change to `pattern.Headroom`, `pattern.ProjectOverview` or `lyx loom lint-pattern`.

## Decisions

### A separate per-card check

Decision: a new blocking check, `pattern-card-headroom`, is one finding per pending card whose own step grows the projected overview past the headroom line, attributed to that card; the whole-plan `pattern-headroom` check stays as it is.
Rationale: the card gate's `lint-pattern` compares the card's working tree with the previous card's commit, so the plan check must measure each card against the overview the cards before it leave; the whole-plan check still catches growth already in the worktree against the merge base, which no pending card causes.

### The running projection

Decision: the projection starts at the worktree's `PATTERN.md` and walks the pending cards in plan order; each card's step applies its fenced entry lines with `pattern.ProjectOverview`, and `pattern.Headroom` measures the step's result against the overview before it.
Rationale: batches run in plan card order and each card lands as its own commit, so the worktree holds every done card's work and each pending card's gate sees the overview the earlier cards leave.

### Crediting a removal to a card

Decision: a `removes` entry of the plan's `## PATTERN changes` is applied in the step of the first pending card whose text names the entry's backticked ID; a removal no pending card names is applied before the first card.
Rationale: the plan ties a removal to no card, and the card that deletes an entry's bullet names it in its prose; crediting an unnamed removal at the start keeps the check from flagging a plan whose removing card is already done or describes the entry another way, so the check can only miss, never invent, a card gate failure.

## Constraints

- `PATTERN-sole-parsers`: card text and fenced entry lines are read only through `planparser`'s parsed `Card` and `CardPatternEntries`.
- `PATTERN-refusal-way-forward`: the new finding names its way forward, and a test reaches it.
- `PATTERN-test-speed`: coverage is rows in the existing table-driven test, `tier` tag unchanged.

## Auto-mode assumptions

- A darn run has no interview; every call here is the writer's self-pick, logged in the support log's question ledger.
- Cards run in plan card order, as `internal/batcher` guarantees and `websterengine.CheckBatchOrder` asserts.

## Open risks

- A card that edits `PATTERN.md` in prose, outside fenced entry lines, is invisible to both checks, as it is to the whole-plan check today.
- A removal credited before the first card makes a card that in truth precedes the removing card look smaller than its gate will measure it.

## Acceptance criteria

- A plan whose earlier card fences a line growing the overview past the line, and whose later card shrinks it back, gets one `pattern-card-headroom` finding on the earlier card and no `pattern-headroom` finding.
- A card whose own fenced lines net to no growth passes, even when an intermediate line within it would grow past the line.
- A card fencing a line that grows the overview, while the same card's prose names an entry the plan removes, passes when the removal frees enough bytes.
- A removal no pending card names shrinks the projection before the first card.
- Every `pattern-card-headroom` finding names its card and the way forward.
- A done card is never projected and never flagged.

## Notes for the plan writer

- `patternPlanFindings` already holds the removed IDs and receives the pending cards as `cards`.
