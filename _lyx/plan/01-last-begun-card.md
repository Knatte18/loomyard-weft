# Card 1 — last-begun-card

**Create:**
- `plan:internal/websterengine#LastBegunCard` -> `func LastBegunCard(st *State) int`

**Edit:**
- `internal/websterengine#TestDispatchScope`

**Intent:**
Add `LastBegunCard` to `internal/websterengine/beginbatch.go`, beside `DispatchScope` and `DoneCards`: it returns the highest card number among the card ids of every batch record `st.Batches` holds, terminal or not, and 0 for a nil state or a state with no batch record.
A record carries its card ids in `BatchState.Cards`; a record written before that field existed names the single card `NN-<Slug>` with `NN` its batch number, exactly as `DoneCards` reads it, and the number is read from an id with `cardNumberInt`.
This is the begun set D2 names, not the terminal-done set `DoneCards` returns, so a batch begun and not yet terminal counts.
It reads the batch records alone and needs no plan and no batchifier, so the renumber verbs of card 3 can bound a position from the run record without forming batches.

Extend `TestDispatchScope`, which already pins the begun scoping over a state holding a terminal batch and a begun non-terminal one: assert `LastBegunCard` over that same state is 2 (the non-terminal legacy record counts), over a state whose only record is non-terminal (`Terminal: false`) and lists `Cards: []string{"04-a", "05-b"}` is 5, so a non-terminal record's own card ids count, and over a nil state is 0.

**ImpactSummary:** One new read-only exported function in websterengine; no existing member changes, and the extended test is the only one touched.
