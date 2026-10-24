# Retagging cmd, contracts and tools under a tier0 failure the batch did not cause

`cmd/lyx`'s `TestCLIForm_EveryCommand` already failed before this batch: the `Long` text of `lyx batten run` (2026 bytes) and `lyx gate test` (2547 bytes) exceeds the 2000-byte cap, as gate run logs from before this batch show.
Since `lyx gate test` stops at the first failing tier, the card gate and the batch gate never reached the guard step or `tier1`, so the retag's `tier1` compile, its vet steps under every tag set and its guard step got no check from either gate.
I ran `lyx gate test --wait --tier 1 --card 23-retag-cmd-contracts-tools` by hand to cover that; it passed.
A later batch hits the same blind spot until a card or the final gate's fixer moves those two `Long` pages into their package docs.
