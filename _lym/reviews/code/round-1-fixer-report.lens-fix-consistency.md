Lens fix-consistency: no findings.

- `cli.go` long help and `doc.go` now say "anchor directory", agreeing with the `render` verb's help in `verbs.go` and the `RenderRequest.Anchor` doc at `render.go:37`.
- A grep over `internal/heddlecli` and `internal/heddleengine` leaves "root" only at `render.go:37`, which is correct for the no-anchor case.
- `RenderWritesFileTargetsAtTheAnchor` matches its siblings' helpers and `Marker` assertion.
- `anchored_hub_wiring_seeds_the_lines_under_the_anchor` mirrors `anchored_hub_seeds_the_lines_under_the_anchor`, reusing `warpExcludeLines` and `countLine`.
- F1 was left as code with a dispute: `hubgeom/webstergeom.go:19-31` sets Webster's `WorktreeRoot` to the anchor path, so `MerriamBase` already reads at the anchor. The `merriambase.go:38` doc comment says "worktree root" for that field, outside this change set.

Tool calls:
1. grep for "worktree root" wording over `internal/heddlecli` and `internal/heddleengine`, excluding tests: to find twins of the wording F3 changed.
