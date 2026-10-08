# Card 16 — amended-cards-rebaseline

**Create:**
- `plan:internal/websterengine#AmendedCard` -> `type AmendedCard struct`

**Edit:**
- `internal/websterengine#BatchState`
- `internal/websterengine#Rebaseline`
- `internal/websterengine#RebaselineResult`
- `internal/websterengine/doc.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/state_test.go`
- `internal/webstercli#websterCLI.rebaselineCmd`
- `internal/webstercli/verbs_test.go`
- `contracts/specs/refusal-spec.md`

**Intent:** `lyx webster rebaseline --card NN` accepts an edit to a card of the in-flight batch, so an operator decision against a running card lands in the card itself (D6's rebaseline half; card 17 makes the batch re-run on it).
`AmendedCard` is one amended card of a batch: `Card` (its `NN-slug` id) and `Rendered` (false when recorded, set true by the first recovery spawn that renders it, card 17); `BatchState` gains `AmendedCards []AmendedCard` (JSON `amendedCards,omitempty`).
`Rebaseline` handles a changed card of a begun, non-terminal batch the way it handles one of a terminal failed, dead or stuck batch: when named with `--card` it restamps the card's recorded hash and records the card in the batch's `AmendedCards` with `Rendered: false`; a card that already has an entry gets that entry's `Rendered` set back to false, so a re-edit while a recovery runs on the earlier text forces that recovery's record failed (card 17).
An unnamed changed card of such a batch keeps the "not named with --card" refusal, and the old "unfinished, run record-batch or recover-batch first" refusal is removed.
`RebaselineResult` reports the amended cards (`cards_amended` in the CLI envelope).
The live attempt, fork or recovery strand, keeps running on the old text; nothing is stopped.
Bound: only the in-flight batch's cards named with `--card`, never a change to a batch's card-ID set (the existing card-set rule still refuses it), and the overview only as card 19 bounds it; a terminal `done` batch's card stays refused.
`rebaseline_test.go` turns the unfinished-batch refusal rows into acceptance rows asserting the restamped hash and the `AmendedCards` entry, and adds a re-edit row resetting `Rendered`; `state_test.go` round-trips the field; the CLI test asserts `cards_amended`.
The rebaseline long help and the package doc's rebaseline section state the in-flight acceptance and that the batch is re-run by recovery; `refusal-spec.md` drops the unfinished-batch reason from the rebaseline card-set row.

**ImpactSummary:** Adds one persisted BatchState field and turns one rebaseline refusal into an acceptance; record-batch and recover-batch read nothing new until card 17.
