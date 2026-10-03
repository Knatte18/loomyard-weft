# Card 12 ordered before the cards it depends on

Card 12 deletes `PairSiblingRemnant`, `RemovePairBranch` and `PairBranchResult`, and its intent says it runs once card 11 dropped batten's use of them.
Webster ordered it fifth, before cards 2, 9 and 11, so `internal/battencli/wire.go` still calls both helpers and `go build ./...` cannot pass after the deletion.
Making it build would mean rewriting card 11's teardown closure against a composite that does not exist yet, so I made no change.
The plan needs card 12 to declare a dependency on card 11 (and card 2).

The cold recovery strand hit the same wall: the recovery reasons only restate the missing deletion, and nothing in the tree has changed since.
Card 2's `Uses:` lists `Topology.RemovePairBranch`, so the plan itself says card 2 reads what card 12 deletes; card 12 must run after cards 2 and 11.
