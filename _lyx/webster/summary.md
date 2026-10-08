# Plan cards name members as glyphs

Both batches recorded `done`.

## Batch 01 — glyphchain-plan-gate (cards 1–6), head `cb415e736`

- Committed the glyph-chain fixture module and the untagged plan-gate legs of the D1 test, plus its integration legs (delta, handle binding, rewrite and done-checks, rework generations).
- A Rename's declaration is now derived for an interface method, and a one-name declaration for a var or const Rename, so both canonicalize.
- Removed `containment-unit-overlap`, `containment-file-overlap` and `handle-unreferenced`.
- Known gap: quarry indexes no struct fields, so `shapes#Struct.Field` resolves `not_found`; the D1 row asserts that answer.
- Deviation: `cmd/lyx/testdata/test-budget.yaml` changed (the `internal/planglyph` row was raised), because the test-budget guard failed otherwise.
- The fork reported that card 1's gate runs `go test -tags integration` on fixture packages inside a nested module, which Go refuses with `[setup failed]`; it ran the equivalent inside the fixture module instead.

## Batch 07 — redundant-package-target (cards 7–12), head `d9d01fa65`

- Added the same-card redundancy checks `redundant-package-target` and `redundant-file-target`, the `**Edit:**` re-sign arrow parse with `resign-not-member`, the re-sign head match with `resign-head-mismatch` and `resign-interface-method`, and `caller-uncovered` caller coverage.
- Updated the Plan-Write stencil and the Plan-Review rubric.
- Deviations: `cmd/lyx/testdata/test-budget.yaml` (budget row raised again) and `internal/planglyph/handle.go` (a `resignHeadMismatch` helper and a `resign` field on `declSource`, beyond `CanonicalizeHandles`).
- The fork reported the same nested-module gate problem for card 11's fixture packages and verified inside the fixture module instead.
