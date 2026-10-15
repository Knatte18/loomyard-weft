# Creel read sets restored to the board's definition

Batch 17 (creel-read-sets) was the plan's only batch and recorded `done` at head `3456a31770af7c05b6d94553a67ecaa6b4debb51`.
The digest listed no deviations, and the scope guard found no files changed outside the batch's targets.

Per the fork's reply, `Store.Read` in `internal/creelengine/verbs.go` now hands out the board's sets:
plain `read` returns the oldest unread message plus every other unread message from its sender,
`read --from` returns every unread message from that sender,
and `read --all` returns every unread message, narrowed to `from` when that is also set.
Only the messages handed out are reserved, so `Ack` and `Release` act on exactly those.
The comments, the `readCmd` help and `doc.go` name the three sets.
Tests: `Read_SetsAndInbox` covers the interleaved sequence, and `TestDaemon_Protocol` swaps its last `read --all` assertion for a new row.

The plan-level verify gate runs after this turn ends.
