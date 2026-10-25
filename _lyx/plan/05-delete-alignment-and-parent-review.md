# Card 5 — delete-alignment-and-parent-review

**Delete:**
- `internal/alignment#`
- `internal/parentreview#`
- `internal/loomengine/parentreview.go`
- `internal/loomengine/parentreview_test.go`
- `internal/loomengine#TestLoomParentReviewAccessors`
- `contracts/stencils#LoomTemplateAlignment`
- `contracts/stencils#LoomFragmentPRGateLoom`
- `contracts/stencils#LoomTemplateParentReviewDelivery`
- `contracts/stencils#LoomTemplateParentReviewBrief`
- `contracts/stencils#LoomTemplateParentReviewKeepalive`
- `contracts/stencils/loom/loom-template-alignment.md`
- `contracts/stencils/loom/loom-fragment-pr-gate-loom.md`
- `contracts/stencils/loom/loom-template-parent-review-brief.md`
- `contracts/stencils/loom/loom-template-parent-review-delivery.md`
- `contracts/stencils/loom/loom-template-parent-review-keepalive.md`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `docs/overview.md`
- `pattern/PATTERN-cwd-resolution.md`

**Intent:** D1 and D2: with their last callers gone in cards 2 to 4, delete the alignment agent and the parent-review store.

- Delete the `internal/alignment` and `internal/parentreview` packages whole, tests included.
- Delete `internal/loomengine/parentreview.go`, the delivery, keep-alive and brief prompt renderers, with its test file, and `TestLoomParentReviewAccessors`; the accessors it pins, which live in `config.go`, go with card 6's rewrite of that file, and card 2 gave `internal/reportengine` its own archived-round path.
- Delete the stencils `loom-template-alignment.md`, the measure fragment `loom-fragment-pr-gate-loom.md` and the three `loom-template-parent-review-*` stencils, their embed vars, their `entries` lines, the `alignment` row of `roleOpeningStencils` and the keep-alive row of `TestStencils_TopLevelMarkers`; the registry test then matches the on-disk tree.
- `docs/overview.md`'s module tree drops the `internal/alignment/` and `internal/parentreview/` lines.
- `pattern/PATTERN-cwd-resolution.md`'s Bound packages list drops `parentreview`.

`go build ./...` and the stale-allowlist checks pass because no production or test file outside the deleted packages imports either one after cards 1 to 4.

**ImpactSummary:** Deletes two packages, one loomengine file, five stencils and their registry entries; no caller remains after cards 1 to 4.
