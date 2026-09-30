# Card 37 — rebaseline-card-hashes

**Edit:**
- `internal/websterengine/state.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/rebaseline.go`

**Uses:**
- `internal/planparser/plan.go`

**Intent:**
Fix PR #321 finding 3: `Rebaseline` compares only the `NN-slug` strings in `BatchState.Cards`, so rewriting the body (targets, Verify) of an executed or in-flight card while keeping its slug is accepted.
That contradicts `refusal-spec.md` ("cards removed or changed") and the verb's help, and a weakened Verify then gives a false pass in `rerunCardVerifies` and the done-checks.

- `BatchState` gains `CardHashes map[string]string` (`json:"cardHashes,omitempty"`), keyed by the same `NN-slug` id `Cards` holds, each value the hex SHA-256 of that card file's bytes (`planparser.Card.SourcePath`).
- A helper beside `batchCardIDs` computes the map for a batch; reading a card file fails as D2's grouped transient I/O row does, before any state mutation.
- `BeginBatch` and the recover-batch record write fill `CardHashes` wherever they fill `Cards`.
- `Rebaseline` compares, for every begun batch, the recorded ids as today and, when the record carries `CardHashes`, each card's recorded hash with the hash of the edited plan's card file of that id.
  A changed hash refuses with the existing `ErrRebaselineCardSetChanged` wrap, naming each changed card as `batch N card NN-slug changed since it was begun`, with the existing restore/reset way forward.
  A record without `CardHashes` (written before this change) keeps today's id-only comparison.
- The `rebaseline` verb's help says a begun card's content counts, not only its id.
- `doc.go`'s rebaseline paragraph states the same.

Tests (`rebaseline_test.go`, the existing begin fixture): after batch 1 is begun and recorded done, editing card 1's `**Verify:**` line while keeping its file name → `Rebaseline` refuses naming batch 1 and the card id, `State.PlanFingerprint` unchanged; editing an unbegun later card's Intent → accepted with batch 1's record intact; a begun batch's record without `CardHashes` whose ids still match → accepted; a recover-batch record carries `CardHashes` for its cards.

**ImpactSummary:** BatchState gains an omitempty CardHashes map filled at begin and recover; Rebaseline refuses a changed begun card body.
