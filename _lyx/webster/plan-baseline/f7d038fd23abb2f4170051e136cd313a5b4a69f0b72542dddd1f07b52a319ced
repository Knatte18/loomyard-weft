# Card 7 — sparse-key-moves

**Edit:**
- `internal/configsync#moveFile`
- `internal/configsync#moveKeys`
- `internal/configsync#moveFile.takeKey`
- `internal/configsync#moveFile.write`
- `internal/configsync#TestReconcile_MovesKeysAcrossModules`
- `plan:internal/configsync#TestReconcileHubWideAt_StripsAnUnmigratedBaseOnce`

**Uses:**
- `plan:internal/configsync#sparseSeed`
- `internal/configsync#lensesFixture`
- `plan:internal/configengine#SparseBaseHeader`
- `plan:internal/configengine#BaseSparseMarker`

**Intent:**
A key move into or out of a hub's base file writes it sparse (D4's key-move bullet).
The hub base is the layer `moveKeys` is told as `configengine.LayerDefault`, the one `ReconcileHubWideAt` passes; a standalone worktree base (`LayerWorktree` with `base` true, from `ReconcileAll` with no board dir) keeps today's template start and delete, since it stays full.

At the default layer:
- `loadTarget` never starts an absent target from the template or from the prime's full copy.
  In a pass that seeds from the prime (`seedDir` set), an absent target takes the sparse prime seed first, `sparseSeed` over the prime's copy with the former-value drop gated on the prime's own `sparse` marker as in card 6, and the moved units are laid over it; a pass that does not seed from the prime starts the target empty.
  `moveFile`'s `fromTemplate` flag, whose only setter was the template start, goes, and with it the `KeepTarget` exception for a target started from the template in the pass.
- `takeKey` overwrites a target unit equal to the template default with the moved value only while `configengine.BaseSparseMarker(dir)` is absent, reported under `Overwritten` as today; once the marker exists such a unit is a pin, kept with the moved value dropped and reported under `KeptTarget`.
  A target unit differing from the template, a prime-seeded one included, is kept and the moved value dropped, as today.
- `write` saves a default-layer file the move left with no unit as `configengine.SparseBaseHeader`, never deleting it, for the target and the source alike; a worktree- or local-layer file emptied by a move is still deleted.
A moved unit equal to the template or a former value is written by the move and dropped afterwards only by the walk's one-time base strip on an unmigrated board (card 6), bar `lenses`' template-equal entries, which every reconcile drops.

`moveKeys`' doc comment states these rules, dropping "an absent target starts from the prime's copy of the target" and "A source file the move empties is deleted" for the default layer.

`TestReconcile_MovesKeysAcrossModules`'s board rows follow: "a repo default moves into a created full claude.yaml" becomes a created claude.yaml holding only the moved `binary` unit; "a base target holding the template default is overwritten" keeps its overwrite with no marker and gains a twin with the board marker present that keeps the target's template value as a pin and reports it kept; a board source the move leaves with no unit is present and holds only `configengine.SparseBaseHeader`; "a board default seeded from the prime's shuttle.yaml carries the moved value" expects the seeded target to hold the prime's non-template units and the moved one only; the lenses rows keep their expectations, since `lenses` was already cut to its tuned entries.
The standalone row "a standalone worktree creates a full claude.yaml from the template" is unchanged and pins that a standalone base still starts full.
The row table gains a `boardMarker bool` field that writes `configengine.BaseSparseMarker` at the board dir before the reconcile, since its `files` keys name module files only.
Two new rows seed an absent board target from the prime: the prime's copy of the target holds a unit equal to its former value, and with the prime unmarked the seeded target drops it, while with the prime's `sparse` marker present it keeps it.

`TestReconcileHubWideAt_StripsAnUnmigratedBaseOnce` (card 6) gains the row that needs this card's empty target start: a key move on an unmigrated board under `StripBase` and `Apply` that carries `shuttle`'s `claude_prompt_cache_ttl_roles`, equal to the former map, into an absent `claude` base leaves that created base present and header-only.

**ImpactSummary:** `moveKeys` at the default layer only, reached from `ReconcileHubWideAt`'s first step; worktree and local moves and run-settings moves are unchanged, and one test's board rows follow.
