# Implementing batch 01: helpers with no caller yet

Cards 1 and 3 each add an exported function whose first caller is a later card,
and the tier1 `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` guard rejects that at the card gate.
I allowlisted each in its own commit with a `Loosened-Check:` trailer.
The `configengine.KeyName` entry left with card 2.
The `internal/yamlengine.MoveKeys` entry is still in `cmd/lym/testonlycode_linux_test.go`:
the card that adds its caller (card 6) must delete it, since a stale entry fails the guard.

Card 3's "moves run in sorted old-key order" is wrong for tapestry's table:
`part_writer` sorts before `parts`, so its new area `parts` is still a scalar.
`MoveKeys` moves the keys that name their own area first and returns the changes sorted by old key.
