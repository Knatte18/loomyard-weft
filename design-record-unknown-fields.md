# Run-record readers accept fields a newer binary added

`record-unknown-fields` · Note · bug, batten, shed, loom

## Seen (2026-10-12)

`plan-gate-fixes`' batten halted at Run-Shed after its child loom run was done, with `battenshed: Run-Shed: read status: state: decode failed: json: unknown field "routed_to"`.
The batten process had started on an older binary than the one the loom run wrote its `status.json` with, and the newer binary had added `routed_to`.
Re-running `lyx batten run plan-gate-fixes` on the new binary landed it.

The batten and loom processes of one run are long-lived and can be on different binaries, since prod moves under them at every landing.
A reader that refuses unknown fields therefore halts a run whenever a landing adds a field to a record it reads.
`board-field-drop` fixes the same class for `board.json` writers, which dropped fields they did not know, but not for readers of other records.

## Want

Every reader of a run record (`status.json` and the other whole-file records batten, shed and loom read across processes) accepts fields it does not know, and every writer of such a record keeps them, as `board-field-drop` does for the board.
A test pins it: a record with an extra field reads and round-trips.
