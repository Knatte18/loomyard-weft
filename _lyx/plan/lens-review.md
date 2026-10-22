# Lens review

Every fork returned a report: affected-set, guard-reach, way-forward, criteria-coverage, branch-coverage, row-reachability, forced-edits, test-economy.

- branch-coverage + affected-set, BLOCKING, a hub worktree file holding `lens_set: ""` (the template's own value, written by `hubreconcile`'s pre-migration fixture and a standalone reconcile) is refused, breaking the migration walk: fixed differently — the proposed fixture change was not made, since a real standalone-reconciled file joining a hub carries the same line and would deadlock the walk that strips it; instead the parent recorded the D3 refinement (refuse only a value differing from the base's) and cards 1 and 2 apply it at load, `Set` and `Edit`, with a card 1 row loading `pick: ""` over a base without `pick`.
- criteria-coverage + forced-edits, MEDIUM, card 1 rewrites two PATTERN entries the record does not declare: fixed as proposed — the parent recorded both as `changes` after Discussion; the overview's `## PATTERN changes` and card 1 carry them.
- criteria-coverage + forced-edits + affected-set, MEDIUM, `fingerprintOf` ignores overlay documents, so a layer-only edit runs no `hubreconcile.Ensure`: fixed as proposed — card 3 hashes `Spec().Overlay` in `fingerprintOf` with two `TestFingerprint` rows.
- criteria-coverage, MEDIUM, nothing pins the `go` layer verbatim after card 4: fixed as proposed — card 3 creates the golden `testdata/go-lenses.yaml` and both cards compare the go layer with it.
- way-forward, MEDIUM, hub-form refusals lack `boardDirWayForwardClause`: fixed as proposed — card 1 ends both hub texts with it and its rows assert `the orch runs it`.
- row-reachability, MEDIUM, card 5's migrate row omits `wantInherited`: fixed as proposed — `wantInherited` `lens_set`.
- row-reachability, MEDIUM, `TestLoad_LayerStack`'s loop overwrites `overlaySpec`'s per-name maps and the standalone row logs a fill: fixed as proposed — the overlay swap runs after the loop's assignments and the standalone row uses `fullDefault` with `wantFill` `lenses`.
- branch-coverage + guard-reach, MEDIUM, `selectorValue`'s non-scalar refusal unreached: fixed as proposed — card 1 row `pick: [x]`.
- branch-coverage, MEDIUM, `Set`'s selector-from-file branch unreached: fixed as proposed — card 2 row with a file holding `pick: x` and `lenses.a=X` alone.
- branch-coverage + criteria-coverage + guard-reach, MEDIUM, card 5's unknown-value `*FileError` unreached: fixed as proposed — three `TestReconcile_FailureNamesItsFile` rows with new `content` and `wantErr` fields.
- guard-reach + row-reachability, MEDIUM, `TestEdit` cannot seed a worktree file's bytes: fixed as proposed — an `existing` row field.
- criteria-coverage, LOW, `Edit`'s `heldBefore` text untested: fixed as proposed — a card 2 row editing a file already holding `pick: x`, asserting `delete the pick line from`.
- criteria-coverage, LOW, the worktree refusal row does not assert the file's path: fixed as proposed.
- branch-coverage, LOW, `stripMovedUnits` and `reconcileSparseFile` get no lenses row: `stripMovedUnits` fixed as proposed with a `TestReconcile_MovesKeysAcrossModules` row; `reconcileSparseFile` disputed, every fix worse — `yamlengine.Prune` never removes a per-name name (`internal/yamlengine/layers.go:333-340`) and an overlay adds only per-name names, so no row can tell the two templates apart.
- branch-coverage, LOW, `TemplateUnder` with an absent base unreached: fixed as proposed — `TestLayers` assertions.
- branch-coverage, LOW, `WarnSkippedEntries` origin unpinned for a stacked `remove`: fixed as proposed — a card 6 row with a board `remove` under a worktree file, asserting the board path and `layer=default`.
- branch-coverage, LOW, `Edit`'s accept exit unreached: fixed as proposed — a card 2 row saving a selector-free edit.
- test-economy, LOW, card 6's list-replace row repeats `TestLoad_LayerStack`: fixed as proposed — dropped, replaced by the origin row above.
- test-economy, LOW, card 1's local-file row repeats `ValidateLocal`'s refusal: fixed as proposed — it asserts the refusal lacks `only the base file`.
- test-economy, LOW, card 3's `lens_set: xyz` row repeats card 1's: fixed as proposed — dropped; card 4's CLI row asserts `go` among the shipped layers.
- test-economy, LOW, card 4 asserts fixed prose: fixed as proposed — only the neutral-base check stays, and the go pin moves to the golden file.
- row-reachability, LOW, `wantSaw` exact against re-marshalled bytes: fixed differently — a `wantSawScaffold` field compares with `BaseScaffold`'s output rather than a literal.
- row-reachability, LOW, `TestSet` cannot assert absent text: fixed as proposed — an `errLacks` field.
- row-reachability, LOW, card 4's default-layer row lacks `wantCommits: 1`: fixed as proposed.
- row-reachability, LOW, card 4's standalone CLI row is tautological: fixed differently — not dropped, since it is the only end-to-end standalone `--set` criterion row; it also asserts each lens differs from the no-file load.
- way-forward, LOW, the removal step has no non-interactive route: fixed differently — no new verb; the refusal says to delete the line from the named file in an editor.
- way-forward, LOW, a prime-seed unknown value names the board file: fixed as proposed — card 5 names the prime's file, with a reaching row.
- guard-reach, LOW, `scaffoldWithoutPerNameEntries`' parse-failure branch cannot occur: fixed differently — kept as a fail-loud wiring guard that panics, rather than a silent fallback.
- guard-reach, LOW, card 1 claims the local check runs first: fixed as proposed — the sentence no longer claims an order.
- forced-edits, LOW, `ValidateFanKeys` and `twillLenses` wrap the emptied-fan refusal with "define the fan": disputed, premise false — defining the fan in `lenses.yaml` with lenses `remove` does not name resolves it, so the wrappers' step progresses (`internal/lenses/fankeys.go` `ValidateFanKeys`, `internal/loomengine/twill.go:38`).
- forced-edits + affected-set, LOW, `EmptiedFans` must stay nil for `TestReconcileAll`'s `DeepEqual`: fixed as proposed — card 6 allocates it only when a fan empties.
- forced-edits + affected-set, LOW, stale docs (lenses `doc.go` Reconcile and Load, configcli `doc.go` `--set` and reconcile, `set.go` and `edit.go` headers): fixed as proposed — cards 2, 3 and 5 reword them.
- forced-edits, LOW, the `//testtiming:keep` reason goes stale: fixed as proposed — card 3 extends it.
