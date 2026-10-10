# Card 6 — edited-done-cards

**Create:**
- `plan:internal/websterengine#EditedDoneCards` -> `func EditedDoneCards(plan *planparser.Plan, st *State, planDir string) ([]string, error)`

**Edit:**
- `internal/websterengine/begun_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#DoneCards`

**Intent:** Answer which done cards were edited since their batch began, for the plan gate (D2).
`EditedDoneCards` sits beside `DoneCards`: for every batch the state records terminal done, it hashes each recorded card's file bytes under `planDir` as `batchCardHashes` does and returns, sorted, the NN-slug ids whose hash differs from the batch's recorded `CardHashes` entry.
A nil state, a record with no `CardHashes`, and an id the plan lacks each contribute nothing; an unreadable card file is a returned error.
It reads the record and the card bytes and never parses the plan, so PATTERN-sole-parsers holds.
A new `TestEditedDoneCards` in `begun_test.go` pins an edited done card, an unedited done card, an unbegun card, a record without hashes, and a nil state; the websterengine row of `test-budget.yaml` rises by one.

**ImpactSummary:** One new exported function with no production caller until card 7; the only edits are a test file and a budget row.
