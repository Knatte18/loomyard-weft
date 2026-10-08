# Webster round 2 — fixer report

Both findings were validated against the code and fixed, one commit each.
`go build ./...`, `go vet` (default and `integration,tmux,llm` tags on `websterengine`), `go test ./...` and `lyx loom lint-comments` pass.

## Fixed

- **F1** (LOW, design) — `003b16411`.
  `stopLeftoverStrand` logged its Warn before `StopStrand`, so a dead leftover logged a kill that never happened, and a live one logged twice.
  The helper and its Warn are deleted; `reclaimEntryTimeStrands`, `BeginBatch`, `recoverSpawn` and `RemoveRecoveryStrands` call `StopStrand` directly.
  Shuttle's stop verb's `shuttle: stopping a live strand` Warn, logged only after reed reports the strand live, is now the single log line for a kill, and a dead strand logs nothing.
  `StrandStopper`'s doc comment states that the stop verb logs the kill at Warn.
  No test changed: the finding is not a coverage gap, and no existing test asserted the removed Warn.
- **F2** (NIT, consistency) — `d00a47585`.
  `internal/shuttleengine/doc.go`'s send paragraph names `turnStartIdleOverride` and `confirmBackoffCap` in place of "ten seconds" and "five seconds".
  The first also now reads "idle on every poll", D1's own wording.

## Disputed

None.

## Deferred

None.
