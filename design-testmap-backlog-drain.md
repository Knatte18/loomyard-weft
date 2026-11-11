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
