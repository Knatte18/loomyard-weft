# The prime dobby leaves a coverage map backlog unbuilt until more landings come

`testmap-backlog-drain` · Note · enhancement, dobby, gate, testkit

## Seen (orch, 2026-11-09)

No coverage map exists yet: `_bolt/.lym/testcover` is absent.
The module has about 170 packages, and the prime dobby builds at most `test_map_batch` (5) maps per refresh,
starting one only after `test_map_every` (5) landings.
At those defaults the first full set of maps needs about 170 landings, so the tasks that wait on the maps wait for weeks.

The decision record of project `redundancy-check` (D6) said the dobby starts a refresh on a tick where the prime HEAD moved,
or where the last refresh's exit record reported stale packages its `--max` cap left unbuilt.
`testmap-dobby-refresh` (landed as `f4ed575e`) made the landing count the only trigger,
so a refresh that leaves packages unbuilt starts no next one.
`lym gate testmap` already writes that count in its exit record.

## Idea

While the last refresh reported stale packages left unbuilt, start the next refresh on the first tick after it ends,
in batches of `test_map_batch`, so a backlog drains on its own and each refresh still holds the gate slot for at most one batch.
Keep the landing count as the trigger once nothing is left.

## Answered by the Sketch `testmap-upkeep-design` (board tidy, 2026-11-11)

The decision record of Sketch `testmap-upkeep-design` (approved) answers this note in its D8:
the prime dobby starts an upkeep pass (`lym gate testmap --upkeep`) when no exit record exists, the prime's HEAD moved, or the last exit record names queued work,
so a backlog drains on its own.
`test_map_batch`, `test_map_every` and the dobby's landing count are retired; a new dobby key `test_map_upkeep` (true by default) switches passes off,
and a migration reads `test_map_every: 0` as `test_map_upkeep: false`.
D9 makes a pass take a slot only while another stays free, and yield to a gate.
The idea above is therefore superseded; the note closes when the build tasks filed from that Sketch land D8.
Do not build the batch-by-batch drain described under Idea.

## Related research (board tidy, 2026-11-11)

The research report of `testmap-upkeep` (landed as `da34600e`, under `docs/research/speed/testmap-upkeep-2026-11-11/report.md`)
compared three proposals for keeping the maps fresh; the Sketch above decided among them.
