# Lens review

Every lens fork returned a report.

- test-economy, LOW, card 1's `TestRenderBatchGate` row repeats the `TestCardGateCommand` subtest through the shared `gateScopeOf`: fixed as proposed; the row and the `internal/websterengine#TestRenderBatchGate` target are dropped, and the Intent says the existing two-card row pins the batch union.
- test-economy, LOW, card 1's handle row in `TestCardTargetDirs` reaches no branch the glyph rows miss: fixed as proposed; the handle row is dropped.
- test-economy, LOW, card 2's generic "same or earlier card" silent row repeats the Create and Custom rows: fixed as proposed; it is now the same-card case, a card whose own Edit target is `internal/new/x.go` carrying the Verify.
- criteria-coverage and row-reachability, MEDIUM, card 4's decode-error row cannot tell a re-read from no re-read: fixed as proposed; the row asserts no `loop.restep` key.
- criteria-coverage and row-reachability, MEDIUM, card 4's `status-missing` fallback to `lyx shed seed` is reached by no row: fixed as proposed; a row with the file removed and `MissingStatusWayForward` empty asserts `lyx shed seed`.
- criteria-coverage, MEDIUM, card 3 has no row pinning that the refusal precedes the slot and the prebuild: fixed differently; the proposal was a `--tags integration` refusals row asserting no stand-in record, which alone cannot tell a refusal before the slot from one after a free acquire. The row also holds the pool's one slot in its setup, so exit 1 rather than `SlotBusyExit` pins the order before the slot wait, and no record pins the order before the prebuild; the table's `args` become a per-row field.
- criteria-coverage, LOW, card 2 has no `go build` row: fixed as proposed; `go build ./nope` is a reported row.
- criteria-coverage, LOW, card 2 has no row for a Verify on the Move card itself: fixed as proposed; the Move row covers the Move card and the card after it.
- criteria-coverage, LOW, card 4's `LoopSpec.ReadStatus` seam is unlicensed and unbounded: fixed as proposed; the Intent states it is a test seam no arming module sets, so production always reads through `internal/state` and nothing is let through.
- criteria-coverage, LOW, card 4's `logger.Warn` at either cause is untested: fixed as proposed; each row captures the log with `logcapture.Capture` (added to Uses) and asserts a WARN line naming the status path.
- row-reachability, MEDIUM, card 2's ancestor row passes without the ancestor rule because `internal` exists on disk: fixed as proposed; the row creates `fresh/new/x.go` and verifies `./fresh`.
- row-reachability, LOW, card 2's overview-chain Move bound is reached by no row: fixed as proposed; a silent row puts `go test ./nope` in the overview chain of a plan with a Move card.
- row-reachability, LOW, card 2's `lyx gate test` parsing of `--tags` and `--` has no row: fixed as proposed; `lyx gate test --tags integration ./nope` reports and `lyx gate test ./internal/x -- ./nope` stays silent.
- forced-edits, LOW, card 4 leaves `internal/shedverbs/doc.go`'s re-step sentence and `loopRestep`'s doc comment stale: fixed as proposed; `internal/shedverbs#loopRestep` joins Edit, both texts are updated, and the Intent states the re-read's record carries an empty `Transient`.
- forced-edits, LOW, card 1 leaves `internal/planparser/doc.go`'s "used only by validate.go's disk-existence checks" sentence stale: fixed as proposed; the Intent corrects that paragraph too.
