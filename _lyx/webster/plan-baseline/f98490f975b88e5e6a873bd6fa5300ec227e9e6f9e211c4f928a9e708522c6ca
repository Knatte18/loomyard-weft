# Card 12 — Wire the parent review onto Discussion-Write

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomrecipe/fixture_test.go`
- `internal/loomrecipe/shape_test.go`

**Create:**
- `internal/loomcli/parentreview_wiring_test.go`

**Uses:**
- `plan:internal/parentreview#GateConfig`
- `plan:internal/parentreview#Store`
- `internal/loomengine#LoomParentReviewDirRel`
- `internal/loomengine#LoomParentReviewDir`
- `internal/loomengine#LoomParentReviewLockDir`
- `internal/loomengine#ParentReviewDeliveryPrompt`
- `internal/loomengine#ParentReviewBrief`
- `internal/loomengine#Config`
- `internal/shedrun#ReadSeed`

**Intent:**
Turn the review on (decision D1) and commit its records with the discussion (D3).

`loom-recipe.yaml`'s Discussion-Write `gates` list gains `parent-review` (`attempts: 1`, `pass_on_cap: true`) after `discussion`, with a comment that `attempts: 0` turns it off and that a reject re-prompts once and the next arrival is let through.
Discussion-Burler's list is unchanged.

`wire` fills `Env.ParentReview`: the `Store` over `LoomParentReviewDir`/`LoomParentReviewLockDir`; the slug; the reviewer from `shedrun.ReadSeed(location, c.runID)`'s `Parent` (empty when the seed has none, a read error being a wiring error); the two discussion paths; `WaitBound` from `ParentReviewWaitMin`; and render seams calling `ParentReviewDeliveryPrompt` and `ParentReviewBrief` with the hub's stencils directory and that slug and reviewer.
`CommitDiscussion`'s pathspec gains `LoomParentReviewDirRel()`, so the round directories land in the discussion commit through `fabricengine.ScopedPathspec` (Fabric Git Invariant).

Bound: the gate entry holds the run only as card 6's `pass_on_cap` entry, and the pathspec adds only the parent-review directory, whose files only `parentreview.Store` writes and which already sits under `_lyx/reviews`, tracked content the per-transition status commit carries; no new class of content reaches weft.

The two loomrecipe fixtures fill `Env.ParentReview` so the embedded recipe still builds.
`parentreview_wiring_test.go` asserts the Discussion-Write row's gate list is `discussion` then `parent-review` with `pass_on_cap`, the reviewer comes from the seed's parent, an absent parent leaves it empty, and the commit pathspec includes the parent-review directory.

**ImpactSummary:** The Discussion-Write row gains a second gate entry and its commit seam one more directory; every other row and seam is unchanged, and a seed with no parent passes the new entry without opening a request.
