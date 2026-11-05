# Card 17 — board-raddle-verb

**Create:**
- `plan:internal/boardengine#Board.RaddleProposal` -> `func (b *Board) RaddleProposal(slug string) ([]byte, bool, error)`
- `plan:internal/boardengine_test#TestRaddleProposal` -> `func TestRaddleProposal(t *testing.T)`

**Edit:**
- `internal/boardcli#Command`
- `internal/boardcli_test#TestCLI`
- `internal/boardcli/doc.go`

**Uses:**
- `plan:internal/boardengine#RaddleStore.Holds`
- `plan:internal/boardengine#RaddleStoreDir`
- `plan:internal/boardengine_test#writeRaddleStore`
- `internal/boardengine/doc.go`

**Intent:** D10's lookup, `lym board raddle <slug>`.
`Board.RaddleProposal` reads, without the lock, the approved `proposal.yaml` of the store directory that `Holds` answers for slug, the Sketch's own slug or any slug it filed; found is false when the store holds neither.
The `raddle` verb of `boardcli`'s `Command` prints that file verbatim with no envelope, as `get --record` prints a record, and errors for a slug the store does not hold, naming `lym report raddle`, which lists every stored Sketch, as the way forward; it reads only, so a run agent may run it.
Since the store outlives prune and refuses a reused slug (card 14), the lookup works after the raddle run's Land pruned the Sketch, and on another machine once its board pulled the store commit.
`TestRaddleProposal` (tier0, stores written by card 14's `writeRaddleStore` beside the proposal file) has rows: by the Sketch slug, by a filed slug, by a filed project slug, by a slug the store does not hold, after the Sketch entry is pruned from `board.json`.
`TestCLI` gains the verb's rows: a held slug printing the proposal, an unheld one erroring.
`boardcli/doc.go` names the verb.

**ImpactSummary:** One read-only verb and board read; no write path changes.
