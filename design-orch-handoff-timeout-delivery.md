# Orch watcher's handoff timeout counts from the post, so a request queued behind a long turn times out

`orch-handoff-timeout-delivery` · Task · bug, orch, creel

Imported from [issue #845](https://github.com/Knatte18/loomyard/issues/845).

## Seen (orch, 2026-10-23, OSL-1046, lyx e634efb9)

The orch watcher posted its soft cycle request #147 at 06:56:27Z while the orch was in a long turn.
Creel delivered it when that turn ended, at 07:06:34Z, and the orch wrote the named note within a minute.
`lyx orch status` then showed `phase: idle`, `cycle_request: null` and `last_abort_reason: "handoff timed out"`, and no cycle ran, so the note was written for nothing.
The handoff timeout counted from the post, and the ten minutes the request waited for delivery used it up.

## Workaround

None by hand; the next soft request, or `lyx orch distill`, asks again.

## Expected

The handoff timeout starts when the request is delivered to the orch (its creel `read` event), not when it is posted, so a request queued behind a long turn still gets the full time.
