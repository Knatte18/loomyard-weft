# Card 2 — A rename or delete of an earlier card's target starts a new batch

**Create:**
- `plan:internal/batcher#renameBoundaries` -> `func renameBoundaries(plan *planparser.Plan, cards []planparser.Card) []int`
- `plan:internal/batcher_test#groupCard` -> `func groupCard(number int, groups ...planparser.TargetGroup) planparser.Card`
- `plan:internal/websterengine#TestRenameBoundary_BatchesCheckAndGateTheirOwnTree` -> `func TestRenameBoundary_BatchesCheckAndGateTheirOwnTree(t *testing.T)`

**Edit:**
- `internal/batcher#costBatcher.Batch`
- `internal/batcher_test#TestCostBatchifier_Limits`
- `internal/batcher/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `plan:internal/planparser#RefCovers`
- `internal/batcher#NewCost`
- `internal/batcher#DiskSizes`
- `internal/planglyph#DoneChecks`
- `internal/testkit/plankit#Repo`
- `internal/websterengine#renderBatchGate`

**Intent:**
Decision D1: card B never shares a batch with an earlier, distinct card A when one of B's Rename old sides or Delete targets covers one of A's targets.

`renameBoundaries`, in a new file `internal/batcher/boundary.go`, returns one entry per card: the index in `cards` of the first later card that covers one of the card's targets, or `len(cards)` when none does.
A card's targets are the `Refs` of its Create, Edit and Delete groups and the `New` side of each Rename pair (`TargetGroup.Pairs`); `Uses` is excluded, and so are Move, Prosa and Custom groups, which D1 does not name.
A later card's covers are the `Old` side of each of its Rename pairs and the `Refs` of its Delete groups.
"Covers" is `planparser.RefCovers(plan, cover, target)`, the one shape decision; this file tests no ref shape of its own.
A card is never compared with itself, so a card whose own Rename old side covers its own earlier target forms no boundary.
Its doc comment says the rule only adds batch boundaries: it never merges batches, reorders cards or moves a card across a dependency, and every grouping batchifier calls it.

`costBatcher.Batch` computes `renameBoundaries(plan, cards)` once and treats a segment `[begin, end)` of two or more cards as infeasible when `boundaries[begin] < end`, inside the existing early `break`.
Extend that break's comment: the inner loop only lowers `begin`, so a longer segment ending at `end` still holds the covering pair and is infeasible too.
Every sentence stating when a segment is feasible gains the boundary condition: `costBatcher.Batch`'s doc comment ("a longer segment needs at most MaxCards cards and a PeakContext …"), the header comment of `internal/batcher/cost.go`, and `internal/batcher/doc.go`'s "a segment of two or more cards is feasible only when …".
A one-card segment stays always feasible.
The identity batchifier is unchanged, since its batches hold one card.
`internal/batcher` is the only grouping site, so a first init and a rebaseline both derive through it (`PATTERN-batcher-registry`), and a recorded partition in `state.json` is never regrouped.

Tests, all over `costSizes` with `Budget: unlimited`, `MaxCards: 5` and `costWeights`:
add `groupCard` to `internal/batcher/cost_test.go`, a card with the given number and target groups, and add rows to `TestCostBatchifier_Limits` asserting `wantSizes`:

- the tender-batch-07 shape: card 1 creates `plan:internal/a#Foo`, card 2 creates `plan:internal/a#Bar`, card 3 renames the pair `internal/a/a.go#` -> `internal/n/a.go#`; sizes `[2, 1]`;
- card 1 creates `plan:internal/a#Foo` and card 2 deletes the member `internal/a#Foo`; sizes `[1, 1]`;
- card 1 creates `plan:internal/a#Foo` and card 2 renames the member `internal/a#Foo` -> `plan:internal/a#Baz`; sizes `[1, 1]`;
- card 1 creates `plan:internal/a#Foo` and card 2 only edits `internal/a#Bar`; sizes `[2]`;
- card 1 edits `internal/a/a.go#Run` and card 2 deletes the file `internal/a/a.go#`; sizes `[1, 1]`;
- card 1 edits `internal/b/b.go#` and card 2 edits `internal/a/a.go#Run` and renames `internal/a/a.go#` -> `internal/n/a.go#` on the same card; sizes `[2]`;
- card 1 creates the unmappable handle `plan:nohash` and card 2 deletes `plan:nohash`; sizes `[2]`;
- card 1 edits `internal/b/b.go#` and card 2 renames `internal/a/a.go#` -> `internal/n/a.go#`, covering nothing earlier; sizes `[2]`.

The identity batchifier and `ExecutionBatches`' recorded-partition path are unchanged and keep their existing tests: `TestIdentityBatcher_Batch`'s `threeCards` row and `TestExecutionBatches`' row "a recorded partition is kept when the size source changes between calls".

`TestRenameBoundary_BatchesCheckAndGateTheirOwnTree`, in `internal/websterengine/cardgate_test.go`, runs the tender-batch-07 shape end to end:
the cost batchifier with an unlimited budget over `batcher.DiskSizes` of the first tree partitions it into `[1 2]` and `[3]`;
`planglyph.DoneChecks` over the first batch's cards against a `plankit.Repo` tree holding `internal/a/a.go` with `Foo` and `Bar` returns no finding, and so does the second batch's against a tree holding the same file at `internal/n/a.go` only;
`renderBatchGate` over each batch against its tree names `./internal/a` and `./internal/n` respectively, each a directory that exists in that tree.

Docs in this commit: `internal/batcher/doc.go` gets a paragraph on the boundary rule under the cost-model paragraph, with D1's cover shapes by reference to `planparser.RefCovers` and its bound (only adds boundaries; a file-level old side over-splits from a package-unit target in that file's directory).
`contracts/specs/loom-plan-spec.md`'s sentence "How cards group into execution batches is not a card property" gains one sentence: a card that renames or deletes what an earlier card targets never shares that card's batch.

**ImpactSummary:** The cost batchifier may form more batches for a plan with a later rename or delete of an earlier card's target; identity and every recorded partition are unchanged.

**Commit:** 2: A rename or delete of an earlier card's target starts a new batch
