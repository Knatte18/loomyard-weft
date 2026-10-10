# Card 13 — rename-old-side-partitioned

**Edit:**
- `internal/planglyph#renameDeclSource` -> `func renameDeclSource(card, oldRef, newHandle, worktreeRoot string, results map[string]quarry.ResolveResult) (declSource, Finding, bool)`
- `internal/planglyph#CanonicalizeHandles` -> `func CanonicalizeHandles(plan *planparser.Plan, planDir, worktreeRoot string, results []quarry.ResolveResult) ([]Finding, bool, error)`
- `internal/planglyph#resolvePass`
- `internal/planglyph/doc.go`
- `internal/planglyph/handle_test.go`
- `internal/planglyph/planglyph_test.go`

**Uses:**
- `plan:internal/planglyph#classifyAmbiguity`
- `internal/planglyph#candidateList`

**Intent:** Keep a Rename's old side Found-only and name the way forward for a partitioned member (decision D6).
`renameDeclSource` keeps deriving the new declaration from exactly one signature, so `multipart` and every ambiguous answer stay `rename-old-unresolved`.
For an `ambiguous` old side `classifyAmbiguity` reports partitioned, the detail names each candidate's file and constraint through `candidateList` and the way forward: a Delete of the old member and one Create declaration of the new member's handle on a card that edits each of the old member's files, or an Edit of each file's body; one declaration head serves every file, since the head only computes the new glyph and partitioned declarations share one glyph, and a Create per file is never the way forward because a second claim on the handle is `handle-collision`.
An unpartitioned ambiguous old side keeps today's detail.
`CanonicalizeHandles` takes `worktreeRoot` and hands it to `renameDeclSource`; `resolvePass` passes its own.
Update the `rename-old-unresolved` bullet of `doc.go`.

Tests: add a partitioned-old-side case to `TestCanonicalizeHandles_RenameOldUnresolved` (`handle_test.go`) over a fixture repository with the member declared in `//go:build linux` and `//go:build !linux` files, asserting the finding names both files, both constraints, the Delete-plus-one-Create route and the Edit-per-file route, and that the card is not rewritten.
Update every `CanonicalizeHandles` call in `handle_test.go` and `planglyph_test.go` for the new parameter.

**ImpactSummary:** One exported and one unexported re-sign whose callers are resolvePass and two test files; the Rename old side's verdict is unchanged, only its detail gains the partitioned way forward.
