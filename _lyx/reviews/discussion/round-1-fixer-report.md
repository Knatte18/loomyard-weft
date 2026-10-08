# Fixer report — discussion round 1

All nine findings fixed; none disputed, none deferred.
`lyx loom validate-discussion` exits 0 after the fixes.

## Fixed

- **F1 (BLOCKING)** — Verified: `ValidateDispatch` calls `planparser.ValidateFormat` and shares `resolvePass` (`internal/planglyph/planglyph.go`).
  D3 now says the pure pairings run wherever `planparser.ValidateFormat` runs, `ValidateDispatch` included, because counting a handle as a member of its declared unit keeps their verdict stable across canonicalization.
  The file-versus-member pairing runs in a separate resolve-backed pass that `ValidateFormat` and `Validate` call and `ValidateDispatch` does not, explicitly not inside `resolvePass`.
  D5's caller coverage runs in that same pass.
  The D3 bound now covers only the file-versus-member pairing.
  The dispatch acceptance criterion is replaced by two pinned cases (handle canonicalizing into a listed file: no finding at either gate; package self glyph plus handle: same finding before and after canonicalization), and a new criterion pins that `ValidateDispatch` reports no caller-coverage finding after the reference set shrinks.
- **F2 (MEDIUM)** — D1 now states every card leaves the suite green, a row lands with the code that makes it pass, and the test card asserts the chain as it stands.
  D7 names which card adds or flips which rows (containment and `rework-generations` in D2's, redundancy in D3's, arrow and caller coverage in D5's).
  New acceptance criterion: every card leaves `go test ./...` and `go test -tags integration ./...` green.
- **F3 (MEDIUM)** — D1 now names the mechanism: the Plan producer drives every shape class through the chain during exploration in an uncommitted probe and writes one fix card per defect, which adds the row the defect breaks.
  A defect the probe missed fails its card's batch, and the way forward is webster's existing failed-card path (extend the card, `lyx webster rebaseline --card NN`, `recover-batch`).
- **F4 (MEDIUM)** — Verified: `LaterDeleteReferences` searches later cards' Edit regions only, and the cross-package pattern qualifies by the package clause name (`deleteorder.go`).
  D5 now says it reuses the per-symbol reference patterns over a new walk of every `.go` file, `_test.go` and build-tagged files included, `testdata` and `vendor` excluded, non-Go files and earlier-card-created files not searched.
  A stated under-reporting bound covers aliased and dot imports and earlier-created files, bounded by webster's per-card build and test gate, with how a planner covers a known one.
- **F5 (MEDIUM)** — D5 now states the arrow's only mechanical effect is marking the member for caller coverage, that nothing downstream compares the head with the built declaration, and that a misstated head is bounded by the Intent and the build and test gate.
  It also states the receiver cases (value to pointer is the same member; a changed receiver type is a mismatch) and that the arrow on a non-member target is a blocking finding; the arrow acceptance criterion gains the file-path case.
- **F6 (LOW)** — The no-double-report sentence and its criterion are scoped to `Delete`; a later-card reference to a renamed or re-signed member is stated to be a caller-coverage finding.
- **F7 (LOW)** — The Goal now states the property the decisions deliver: no check changes verdict between the plan gate and dispatch because of state the run changes, and such checks run at the plan gates only.
- **F8 (NIT)** — D2's bound now says webster runs batches one at a time, each to one fork working its cards in plan order, and that `recover-batch` re-runs the same batch's cards.
- **F9 (NIT)** — Dropped the first and third Auto-mode bullets and the Notes bullet on webster sequencing.
  Cut the restated half of the first Open risk, the restated clause of the second, and the third (a restatement of D1's quarry-row rule) entirely.

Support log: Interview item 3 and Q2 updated to the refined D3 split; Q12–Q15 record this round's self-picks; four rejected alternatives added for the seam, the fix mechanism and the head check.

## Disputed

None.

## Deferred

None.
