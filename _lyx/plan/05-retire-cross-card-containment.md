# Card 5 — retire-cross-card-containment

**Delete:**
- `internal/planparser/containment.go`
- `internal/planparser/containment_test.go`
- `internal/planglyph/containment.go`
- `internal/planglyph/containment_test.go`

**Edit:**
- `internal/planparser/validate.go`
- `internal/planparser/shape.go`
- `internal/planglyph#resolvePass`
- `internal/planglyph#ValidateDispatch`
- `internal/planglyph#resolveTargets`
- `internal/planglyph#cardOwnHandles`
- `internal/planglyph/doc.go`
- `internal/planglyph/status_enforcement_test.go`
- `internal/planglyph#TestValidate_UnparseablePlanDirectoryIsAnInfrastructureError`
- `internal/websterengine#BeginBatch`
- `internal/planglyph/glyphchain_test.go`
- `internal/planglyph/glyphchain_integration_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Intent:**
Retire both cross-card containment refusals (D2): `containment-unit-overlap` (`planparser`'s syntactic tier) and `containment-file-overlap` (`planglyph`'s resolve tier).

- Delete both `containment.go` files and their tests.
  `validate` in `validate.go` stops calling `syntacticContainment`, the file's header list drops the ID, and `checkGlyphMalformed`'s doc comment stops listing `containment-unit-overlap` among the checks that skip a malformed glyph.
  `shape.go` drops `gateSyntacticContainment` and its ledger entry, and the ledger's doc stops counting its policies.
- `resolvePass` stops calling `resolveContainment`, and its doc, `ValidateDispatch`'s doc ("never paired against a pending card for containment"), `resolveTargets`' consumer list and `cardOwnHandles`' "both containment tiers" stop naming it.
- `doc.go` drops the `containment-file-overlap` bullet and names the passes that raise `glyph-rejected` without counting them.
- `status_enforcement_test.go` drops the `containment.go:resolveContainment` allowlist row; `TestValidate_UnparseablePlanDirectoryIsAnInfrastructureError` drops the ID from its resolve-backed list.
- `BeginBatch`'s comment on the passes that run after the rewrite stops naming containment.
- The spec drops the `containment-unit-overlap` row, removes the ID from the `glyph-malformed` row's list of checks that skip a malformed glyph, renumbers the validation-checks list, and fixes every `row N` cross-reference to the new numbers.

Flip the D1 rows: a member and its file, a member or file and its package, on different cards, now validate clean; the `rework-generations` row reports no blocking finding at `ValidateDispatch` after card 1 binds its handle.

What this lets through: two cards whose targets overlap physically are no longer refused at any gate.
What bounds it: webster runs batches one at a time in one worktree, each batch to one fork that works its cards in plan order; batches come back in card order, `CheckBatchOrder` asserts that order, and `recover-batch` re-runs the same batch's cards rather than pairing different ones.
The checks guarded against blind parallel dispatch, which webster no longer does, and the resolve tier changed verdict mid-run because a `plan:` handle has no file until it is built.

**ImpactSummary:** Removes two check IDs and their passes from both validation layers; every gate that ran them now admits overlapping targets on different cards, and no caller outside planparser and planglyph reads either ID.
