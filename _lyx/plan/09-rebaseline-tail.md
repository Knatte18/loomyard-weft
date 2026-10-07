# Card 9 — rebaseline keeps begun batches and re-batches the tail

**Edit:**
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/rebaseline_git_test.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/rebaseline.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/websterengine#PartitionBatch`
- `plan:internal/websterengine#CheckBatchOrder`

**Intent:**
An accepted rebaseline keeps every begun batch as recorded and re-batches only the cards after the last begun batch with the then-active profile (Decision 11).

`RebaselineDeps` replaces `Batches` with `Active batcher.Batcher` and `Sizes batcher.SizeSource`; webstercli passes `c.batcher` and `batcher.DiskSizes(c.geom.WorktreeRoot)`.

**With a recorded partition.**
1. A recorded batch is begun when `State.Batches` holds a record under its number; the kept prefix is every recorded batch up to and including the last begun one.
2. The current plan's first cards must be exactly the kept prefix's recorded card ids, in order; a plan that gained, lost or reordered a card at or before the last begun batch's last card refuses with the existing `ErrRebaselineCardSetChanged` refusal and its way forward.
3. The begun cards' content-hash checks and the `--card` restamp rules run as today over the kept batches, now mapped from the partition rather than recomputed.
4. The tail, every plan card after the prefix, is batched by `Active` against `Sizes`; each tail batch records the active profile's name and its estimate.
5. `CheckBatchOrder` runs over the kept batches followed by the tail.
6. Only once every check passes is `State.Partition` replaced by the kept batches, with their recorded card ids, profile names and estimates, followed by the tail's.
Cards added, removed or reordered after the prefix are accepted and re-batched, each changed plan file named with `--card NN` as the changed-file guard already requires.

**Without a recorded partition** (a state from before card 8), the tail stays on `batcher.Identity()`, the checks run against the identity batches as today, and no partition is recorded, so the run stays on identity until it ends.

What it lets through, and its bound: only unbegun cards regroup, and only at a rebaseline the operator or Master triggers; a refusal leaves `State`, `Partition` included, unchanged; a profile that does not load is refused earlier, by card 6's prologue refusal.

`webstercli/rebaseline.go`'s help text says the unbegun tail is re-batched by the active profile.

**Tests** (`rebaseline_test.go`, `rebaseline_git_test.go` adjusted to the new deps):
an accepted rebaseline keeps each begun batch's card ids, profile and estimate and re-batches the tail with a grouping batcher's profile and estimates;
a card added after the last begun card is accepted and grouped into the tail;
a card added, removed or reordered at or before the last begun batch's last card refuses with `ErrRebaselineCardSetChanged`, leaving `Partition` unchanged;
a state without a partition re-batches with identity and records none.

`doc.go`'s rebaseline paragraph describes the kept prefix and the tail re-batch.
`refusal-spec.md`'s `rebaseline card set changed` row's trigger adds a plan that gained, lost or reordered a card at or before the last begun batch's last card.

**ImpactSummary:** `Rebaseline` and its one CLI caller; its existing refusals and their texts are unchanged, and it now also replaces the recorded partition's unbegun tail.
