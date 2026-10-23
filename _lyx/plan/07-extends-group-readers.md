# Card 7 — Extends group readers outside planparser

**Edit:**
- `internal/planglyph#collectEditTargets`
- `internal/batcher#cardTargets`
- `internal/planglyph#TestLaterDeleteReferences`
- `internal/batcher_test#TestCostBatchifier_Limits`
- `internal/planglyph/doc.go`

**Uses:**
- `internal/planparser#CardTypeExtends`

**Intent:**
D3: every reader of an Edit group's targets reads an Extends group alike, in `internal/planglyph` and `internal/batcher` too.

`collectEditTargets`, which feeds the delete-order check's edit regions (`delete-before-reference`), collects the member-glyph and file refs of Extends groups as it does Edit groups', so code a later Extends card adds to that names a member an earlier card deletes is flagged as an Edit card's would be.
`batcher.cardTargets` returns an Extends group's refs beside Create, Edit and Delete refs, so a later card's rename or delete of an Extends target still forces a batch boundary.

Every other reader in these packages already reads Extends refs: the resolve pass, `DetectDrift`'s references, the scope guard and the batcher's estimate walk the flat `Targets` or every group; the Delete- and Rename-only readers rightly skip it.
`internal/planglyph/doc.go` states that the delete-order regions cover Extends groups.

Tests:
`TestLaterDeleteReferences` gains a row whose later card names under `**Extends:**` a function that calls a member an earlier card deletes, expecting the same blocking `delete-before-reference` finding as the existing Edit row.
`TestCostBatchifier_Limits` gains a row where an earlier card's Extends target is covered by a later card's Delete, expecting the boundary an Edit target gets.

**ImpactSummary:** two unexported group readers widen to one more label; their callers (`LaterDeleteReferences`, `renameBoundaries`) are unchanged and no existing plan carries an Extends group.
