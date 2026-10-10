# Webster round 2 — fixer report

Every finding was validated against the code and fixed, one commit each.
`lyx gate test` over `./internal/shuttleengine`, `./internal/websterengine`, `./internal/planparser` and `./internal/testkit/envkit` is green, `gofmt -l` is clean over the touched packages, and `lyx loom lint-comments` passes.

## Fixed

- **F1** (`02bc55ec7`) — confirmed unreachable: claudeengine parses a Stop with outstanding tasks as `EventWaiting`, which the loop never holds, and a Stop without them is folded by `judgeTurnEnd` as idle-stalled or idle-done, never busy/background.
  `loopAgreesWithState` drops `busyOnBackground` from `loopHeld`, and its comment names the two remaining pairings, with the done reason only.
  `doc.go` drops "an expired-shell turn end" from the hold passage and the held-after-expiry pairing, now two pairings.
  The `waitshadow_test.go` row renamed in card 22 repeated the first row's `busy/background waiting` and is deleted along with `shadowTranscriptShell`, which only it used.
  Card 22 asked for that row to be reworded to a held turn end beside busy background work; with the pairing unreachable, no fixture can build it, so the row goes rather than asserting something the code cannot produce.
- **F2** (`90e4a5f37`) — the joined `waiting_test.go` rows are split, the `entryLine(limit, …)` calls in `validate_test.go` get their spaces, and `envkit.go`'s `nilLegal` takes gofmt's alignment.
- **F3** (`0d446a7c8`) — confirmed: `ValidatePlan` passes `LoadState`'s nil state straight to `EditedDoneCards`, which `TestNewPlanGate/NoRunRecordReportsNothing` reaches.
  The nil-state assertion is dropped and the doc comment names the recorded-id case alone.
- **F4** (`ad1657042`) — the `waiting with output files is done` row is deleted.
  With it gone, `touchOutput`, `wantFirst` and `checkOffset` were constant across the remaining rows, so all three fields go and their assertions run unconditionally; the doc comment and the `testtiming:keep` line drop the done case.

## Disputed

None.

## Deferred

None.
