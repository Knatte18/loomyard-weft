# Implementing the pusher target kinds and the spawn change

Card 7 states that `TeardownSlug`, `PauseSlug` and `ParentLocked` are read only by the push child.
`PushPairAnchored` also read `TeardownSlug` off `statusPushRequest` for its teardown lock and takeover read,
so it now derives the pair's slug itself, outside the card's targets.
A takeover subtest that replayed a spawned request through `CoalescePushRequest` had to go for the same reason.

Card 6 asks the board pusher to yield to a pause through the gate,
but the gate only holds a process `cmd/lym` reads as a detached child, so `--board` had to join that read.
A removed bare repository reads as class "other", not transport; tests of an owed push need a refused URL.
