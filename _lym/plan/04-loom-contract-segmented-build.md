# Card 4 — loom-contract-segmented-build

**Edit:**
- `internal/shedrecipe#LoomBuildEngines`
- `internal/shedbuild#checkLoomContract`
- `internal/shedbuild#TestBuild_LoomContract`
- `internal/shedbuild/doc.go`

**Extends:**
- `internal/shedrecipe/doc.go`

**Uses:**
- `internal/shedbuild#onlyRowOf`
- `internal/shedbuild#onDonePath`

**Intent:**
Let the loom contract accept a build row inside a review segment (D10).

`LoomBuildEngines` gains `HardenFix`, so a recipe holding two `HardenFix` rows, or a `HardenFix` row beside another build row, is refused by the existing one-build-row clause; the engine choice text the refusal names gains `HardenFix`.

`checkLoomContract` changes only for a build row that carries a segment:
- When no row of that segment runs `Bouncer`, it refuses: `the build row %q sits in segment %q, which holds no Bouncer row; way forward: add the segment's Bouncer row`.
- When no walk from the segment's Bouncer over the on_done and on_stuck edges of the segment's rows reaches the build row, it refuses: `the build row %q is reached by no on_done or on_stuck edge from its segment's Bouncer %q; way forward: add the edge from a row of the ring to the build row`.
- Otherwise the order clause reads the build row's place on the on_done path from the entry as its Bouncer's place, so a build row lies after the record row when its Bouncer does, and the existing order refusal fires when the Bouncer lies before the record row.
A build row with no segment is checked exactly as before; those two refusals are the rule's whole reach beyond today's check.

`TestBuild_LoomContract` (tier0) gains rows over a ring of three rows in one segment, Bouncer, review and a `HardenFix` fix row, the Bouncer on the on_done path:
a valid ring passes; a segment with its Bouncer row recast to `Stub` is refused naming the missing Bouncer; a fix row whose ring edges are cut is refused naming the edge; a Bouncer placed before the record row on the on_done path is refused by the order clause.
The existing rows, whose build row carries no segment, keep their expectations; the two engine choice strings gain `HardenFix`.
The shedbuild package doc states the segmented form of the order clause and its two refusals.

**ImpactSummary:** One engine name added to a shared list and one new branch of the contract check, taken only by a build row carrying a segment; every shipped recipe's build row carries none.
