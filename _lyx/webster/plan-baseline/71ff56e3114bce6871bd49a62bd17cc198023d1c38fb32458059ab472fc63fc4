# Card 1 — begun-card-bound

**Create:**
- `plan:internal/websterengine#LastBegunCard` -> `func LastBegunCard(st *State) int`

**Edit:**
- `internal/websterengine/begun_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#DoneCards`
- `internal/websterengine#cardNumberInt`

**Intent:** Add `LastBegunCard`, beside `DoneCards` in `beginbatch.go`: the highest card number among the cards of every batch record `st.Batches` holds, terminal or not, which is the frozen bound the card verbs of card 4 refuse at or below.
A record's cards are its `Cards` ids, read with `cardNumberInt`; a record written before `Cards` existed names the single card numbered by its map key, as `DoneCards` reads it.
A nil state, or one with no batch record, returns 0, so every position is open.
It reads the records alone and needs no batchifier, matching the begun set `DispatchScope` derives, not the terminal-done set `DoneCards` returns.

Add `TestLastBegunCard` to `begun_test.go`, table-driven: nil state, no records, a record with `Cards`, a record without `Cards` falling back to its key, and a done record beside a later non-terminal one, which bounds at the later one.
`internal/websterengine` is at its test budget, so raise its row in `test-budget.yaml` by one.

**ImpactSummary:** One new reader of `State.Batches` beside `DoneCards`; no existing caller or record field changes.
