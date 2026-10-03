# Second recovery of card 12, still ordered before cards 2, 9 and 11

This is the fourth attempt at card 12, and HEAD is still at `8f03fa43f`, the commit the first attempt started from.
`internal/pairteardown` (card 9) and `ErrPairNotFound` (card 2) don't exist.
`internal/battencli/wire.go:411` and `:428` still call `PairSiblingRemnant` and `Topology.RemovePairBranch`, and `wire_test.go:518` calls `PairSiblingRemnant`, because card 11 hasn't run.
Deleting the three symbols breaks `go build ./...`, and batten would lose its only way to finish a half-removed pair.

Card 12's file has no dependency field, and webster ordered its batch right after 08.
Each recovery reason only repeats the delete-not-done check, so another recovery strand will fail the same way.
Fix the batch order (run 2, 9 and 11 first), or give card 12 a declared dependency on cards 2 and 11.
