# Card 3 — webster-last-begun-card

**Create:**
- `plan:internal/websterengine#LastBegunCard` -> `func LastBegunCard(st *State) int`
- `plan:internal/websterengine#TestLastBegunCard` -> `func TestLastBegunCard(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#State`
- `internal/websterengine#BatchState`
- `internal/websterengine#cardNumberInt`

**Intent:**
Add `LastBegunCard`, in `internal/websterengine/beginbatch.go` beside `DispatchScope` and `DoneCards`, returning the highest card number among the card ids of every batch record `st.Batches` holds, terminal or not (D2's begun set, not `DoneCards`' terminal-done set).
A record's ids are its `Cards`, or for a record written before that field existed the single id `NN-<Slug>` of its batch number, as `DoneCards` reads them; numbers come from `cardNumberInt`.
A nil state, or one with no batch record, returns 0, so every position is open with no run record.

`TestLastBegunCard`, in `internal/websterengine/begun_test.go`, covers in rows: a nil state (0); a state with no batches (0); a terminal done batch of cards 1 and 2 beside a begun, non-terminal batch of card 3 (3, so a begun and not yet terminal batch counts); a legacy record with no `Cards` keyed 4 with slug `x` (4).

`cmd/lyx/testdata/test-budget.yaml` raises `internal/websterengine` by one.

**ImpactSummary:** additive: one exported read-only function over webster's state, no existing caller changed.
