# Card 8 — one-repository-anchor-change

**Create:**
- `plan:internal/fabricengine_test#TestAnchorChange_OneRepository` -> `func TestAnchorChange_OneRepository(t *testing.T)`

**Edit:**
- `internal/fabricengine#moveWeftContent`
- `internal/fabricengine#AnchorChange.Apply`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#AnchorChangeTrailerKey`
- `internal/fabricengine#oneRepositoryAt`
- `internal/fabricengine#weftAnchorStop`
- `internal/hubforge#NewOneRepoHub`
- `internal/hubforge#AddPairWith`

**Intent:** D5's anchor change in a one-repository hub: one records commit that moves `_lym` from the old anchor to the new one and rewrites `.lym-anchor`.

`moveWeftContent` adds an `AnchorChangeTrailerKey` trailer naming the new anchor to its commit in every layout; the commit stays the one `git mv` plus marker commit it is today.
In one repository that commit lands on the code branch, which card 9's marker-change refusal admits by that trailer.
`AnchorChange.Apply` of the move kind in a one-repository worktree runs its unwire and re-wire steps through the one-repository wiring (no junction).
The integrate kind, chosen when origin's tip carries another anchor, gets its one-repository path in card 13, which replays the local records-only commits onto origin's tip.
The anchor-change stop (`weftAnchorStop`) holds as today: it compares the marker at the branch head and its upstream.

`TestAnchorChange_OneRepository` (tier1, `NewOneRepoHub` at `.`, a pair through `AddPairWith`) rows:
`MovesRecordsAndMarkerInOneCommit` (one commit carrying the trailer, every changed path a records path of `RecordsPathsAt`, no code path changed, the pair resolves at the new anchor);
`OriginAtAnotherAnchorStops` (a second clone moves the anchor and pushes; the first pair's pull stops with the anchor-change stop).

**ImpactSummary:** The anchor-change commit gains a trailer in both layouts and Apply a one-repository wiring branch; trailer readers see one more key and ignore it.
