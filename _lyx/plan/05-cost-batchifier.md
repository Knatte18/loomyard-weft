# Card 5 — cost batchifier

**Create:**
- `internal/batcher/cost.go`
- `internal/batcher/cost_test.go`
- `plan:internal/batcher#CostParams` -> `type CostParams struct`
- `plan:internal/batcher#NewCost` -> `func NewCost(name string, p CostParams) Batcher`

**Edit:**
- `internal/batcher/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `plan:internal/batcher#SegmentCost`
- `plan:internal/batcher#Weights`
- `plan:internal/batcher#SizeSource`

**Intent:**
The cost-model batchifier: hard limits and an exact dynamic program over contiguous segments (Decisions 6 and 7).
Nothing selects it until card 6 wires profiles, and the template keeps identity active.

`CostParams` holds `AloneAbove float64`, `Budget float64`, `MaxCards int` and `Weights`; `NewCost(name, p)` returns a `Batcher` whose `Name()` is the profile name.
Its `Batch`:
1. Computes each card's one-card cost with `SegmentCost`.
2. Runs the DP over the card sequence: `best[i]` is the minimum total cost of splitting cards `1..i`, over the last segment `j+1..i` of at most `MaxCards` cards, so `O(n · MaxCards)` segment evaluations.
   A one-card segment is always feasible.
   A segment of two or more cards is feasible only when no member's one-card cost exceeds `AloneAbove`, it holds at most `MaxCards` cards and its `SegmentCost` stays within `Budget`.
   Ties go to the split with more batches.
3. Returns the batches in card order, each with `Profile` set to the name and `Estimate` to its `SegmentCost`.

What this lets through, and its bound: cards share a fork only inside a contiguous segment of the given order, so order and every forward dependency are kept; a card over `Budget` or `AloneAbove` runs alone exactly as under identity, so no card costs more than today; `AloneAbove` above `Budget` is accepted and changes nothing for such a card.
No value of the thresholds makes the split infeasible.

`cost_test.go` (untagged, fake size source) pins on every input of a table:
card order kept and batches contiguous;
a card above `AloneAbove` alone;
no batch over `MaxCards` or `Budget`;
a card over `Budget` but under `AloneAbove` alone, with the split still returned;
the DP total equal to a brute-force minimum over every contiguous split of small inputs, with a deterministic input set;
ties resolved to more batches;
two small cards editing the same file grouped, two small cards in unrelated packages not;
each batch's `Estimate` equal to `SegmentCost` of its cards.

`doc.go` describes the hard limits, the DP and the tie rule.
`loom-plan-spec.md`'s card-type table loses the `Batchable?` column and its composition-rule bullet, and points to `internal/batcher`'s package doc for how cards group (Decision 13).

**ImpactSummary:** `contracts/specs/loom-plan-spec.md` loses a column no code enforces; the batchifier is additive and unselected.
