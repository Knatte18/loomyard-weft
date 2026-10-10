# Card 6 — edited-done-cards

**Create:**
- `internal/websterengine#EditedDoneCards`

**Edit:**
- `internal/websterengine/begun_test.go`
- `internal/websterengine/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#DoneCards`

**Intent:** Answer which done cards were edited since their batch began, for the plan gate (D2).
`EditedDoneCards` sits beside `DoneCards`: for every batch the state records terminal done, it hashes each recorded card's file bytes under `planDir` as `batchCardHashes` does and returns, sorted, the NN-slug ids whose hash differs from the batch's recorded `CardHashes` entry.
A nil state, a record with no `CardHashes`, a batch that is begun but not done, and an id the plan lacks each contribute nothing; an unreadable card file is a returned error.
It reads the record and the card bytes and never parses the plan, so PATTERN-sole-parsers holds.
`doc.go`'s plan-gate passage names `EditedDoneCards` beside `DoneCards` as what the gate reads.
A new `TestEditedDoneCards` in `begun_test.go` pins what the gate rows of card 7 do not: an edited card of a begun but not-done batch reporting nothing, a recorded id the plan lacks reporting nothing, and a nil state; the edited, unedited, unbegun, no-record, no-hash and unreadable-card-file cases are pinned once, through `TestNewPlanGate` in card 7.
The websterengine row of `test-budget.yaml` rises by one.

**ImpactSummary:** One new exported function with no production caller until card 7; the only edits are a test file, the package doc and a budget row.
