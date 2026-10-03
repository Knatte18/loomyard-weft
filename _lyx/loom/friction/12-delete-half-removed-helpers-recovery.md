# Recovering card 12 before the cards it depends on

This is the third attempt at card 12 that hit the same wall, and the tree at `8f03fa43f` is unchanged since the first.
`internal/battencli/wire.go:411` still calls `fabricengine.PairSiblingRemnant` and `wire.go:428` still calls `Topology.RemovePairBranch`, because card 11 has not run.
`internal/pairteardown` (card 9) and `ErrPairNotFound` (card 2) do not exist either, so card 11's rewrite can't be done from here.
Deleting the helpers anyway would break `go build ./...`, and with card 2 missing it would also remove batten's only way to finish a half-removed pair.

Spawning recovery strands again won't fix this, because the recovery reasons only restate the missing deletion.
The batch order has to change: card 12 must run after cards 2, 9 and 11.
Card 2's own `Uses:` lists `Topology.RemovePairBranch`, so the plan already records that dependency.
