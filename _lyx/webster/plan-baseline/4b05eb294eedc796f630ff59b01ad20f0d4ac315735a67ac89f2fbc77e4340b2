# Card 11 — manifest-content-moves

**Prosa:**
- `internal/treadleengine/doc.go`
- `internal/fabricengine/doc.go`
- `internal/fslink/fslink.go`
- `internal/websterengine/doc.go`
- `internal/burlerengine/doc.go`
- `internal/boardengine/board.go`

**Create:**
- `internal/orchcli/doc.go`

**Uses:**
- `manifest/roadmap.md`
- `manifest/HANDOFF.md`
- `manifest/designs/hardener.md`
- `manifest/designs/fabric-windows-verification.md`
- `manifest/designs/webster-parallel-execution.md`
- `manifest/designs/curation-triage.md`
- `manifest/designs/raddle.md`
- `manifest/designs/reed-mailbox.md`
- `manifest/designs/reed-multi-window.md`
- `manifest/designs/semantic-index.md`
- `manifest/designs/warp-visibility.md`

**Intent:** Move `manifest/`'s surviving content to the board and into package docs, ahead of card 12 deleting the folder (D9).

Board notes, through `lyx board upsert` only (upserts by slug, so a re-run rewrites rather than duplicates):
- For each Someday item not already on the board, add a tier-3 note: `raddle`, `webster-parallel-execution`, `reed-multi-window`, `warp-visibility`, `baseline-drift-gate`, `config-overlay`, `fabric-reset-to-sha`, `finalize-discrepancy-doc`, `shedrecipe-capabilities`, `doctor`, `session-sync`.
  Each note has a `[label]` brief prefix and the item's `roadmap.md` text.
  Add the content of its `manifest/designs/<name>.md` when one exists and describes unbuilt design.
  Check each slug with `lyx board get` first.
- For `hardener`, `semantic-index` and `creel` (from `reed-mailbox.md`), which are already on the board, append the design doc under a `## Design` section of that entry's body.
- Touch no other board entry.

Package docs (semantic line breaks, prose only, no code change):
- `internal/orchcli/doc.go`, new, holding the package doc comment: the hub run loop from `manifest/HANDOFF.md`.
  The rest of `HANDOFF.md` is dropped.
  If `orchcli` already carries a `// Package orchcli` comment in another file, move it here.
- `internal/treadleengine/doc.go`: `hardener.md`'s text-review versus behaviour-review distinction and its respawn rationale.
- `internal/fabricengine/doc.go` and `internal/fslink/fslink.go`'s package comment: `fabric-windows-verification.md`'s Linux-only caveat.
- `internal/websterengine/doc.go`: `webster-parallel-execution.md`'s case study and index-race rationale.
- `internal/burlerengine/doc.go`: `review-finding-classification.md`'s convergence case study, read from its git history if the file is not under `manifest/designs/`.
- `internal/boardengine/board.go`'s package comment: one line, "anyone adds notes, the orchestrator curates".
  The `curation-triage.md` automation is dropped.

Prose moved into Go comments follows the fabric-vocabulary rule for its package: non-owner packages name no side.
`manifest/` is not modified here.
