# lym gate testmap: plan each map at HEAD, keep what holds, check it

`lym gate testmap` no longer rebuilds a package's coverage map once it is stale by age or by a changed file.
It now plans every map at the prime's HEAD `H`, keeps the entries that still hold, and measures only the tests that need it.
The age limit goes with the old rule, and a self-check and an audit round replace it as the guard against drift.

## What changes

- **Plan at `H`.**
  Each map gets the selector's verdict at `H` as target, through `fabricengine.Fabric.CodeFileAt` (`internal/gatecli/testmapplan.go`).
  - *Current* (no affected test, no miss): the map is re-stamped to `H` with every entry kept.
  - *Affected tests*: the affected entries are dropped, the map is written at `H` at once, and the dropped tests are queued.
  - *Miss*: the map is left as it is and the whole package is queued for a rebuild.
    The rebuild's first written round starts from an empty map, so the map at `H` holds only entries measured at `H`.
  - Except under a miss, a declared mapped test with no entry is queued, so gaps left by an ended or discarded pass are filled at the next plan.
  - The plan builds and runs nothing and holds no slot.
    It rewrites a map only under the package's build lock and skips a map whose lock is held.
- **Outcome entries and spawn verdicts.**
  A run that fails, times out, skips or covers nothing records an outcome entry with no coverage.
  A Current verdict keeps it, and a miss or a change to one of the package's own files drops it and queues the test.
  A test the spawn scan marks out-of-process or unresolved is recorded with that verdict, is never run, and counts as an entry, so it is never queued.
  A run ended by a yield, a stop or a signal records nothing.
- **A failed build keeps the old map.**
  The map stays at its `Commit` with its entries and marker, and the failure and `H` go in a new field beside the entries.
  A package with no map, or none with a test entry, gets the failure-only map as before.
  A package is not queued again while its recorded failure names the current HEAD.
- **Self-check and audit.**
  After every build at `H`, a kept entry whose block keys or linked functions are absent from the fresh build is re-measured and counted.
  Each pass ends with one audit round, when it can take a slot at once (`internal/gatecli/testmapaudit.go`).
  The audit re-measures a fixed number of passing entries, oldest measurement first, and counts a result that differs from the kept one.
  An equal result keeps the coverage and sets the entry's measurement time to now, so the audit rotates through the entries.
- **`test_map_max_age_days` is retired** from `internal/gateslot` as a key of its own.
  A hub or machine file that still sets it loads, and reconcile prunes it.
- **`gateslot.Pool.TryAcquire`** takes a slot only when one is free at once and none while a multi-slot acquire drains, for the audit.
- **Redundancy step** counts only passing entries as covers and judges a map with a failure field by its entries.
- **`testcover.ExitRecord`** replaces `Unbuilt` with the planned HEAD, the queued count, the plan's counts of current maps, affected maps and misses by reason,
  the self-check and audit mismatch counts, and per measured package its build seconds, run seconds and test count.
  The verb's report prints the same.
  The record's reads accept unknown keys, so an old record holding `unbuilt` still reads.

## Design decisions

- A carried entry is wrong only where the selector would already wrongly skip that test in a gate, so upkeep adds no new way to be wrong.
  The `finalize` landing gate at scope `all` backstops both.
- A map with gaps is valid, because the gate scope runs every test its map does not record.
- The audit runs once per pass, so on a day with no landing it re-measures nothing.
- Upkeep stays in the prime worktree.

## Not in this change

- The dobby's read of the exit record, and starting a pass from its planned HEAD and queued count, belong to `testmap-upkeep-dobby`.
- `test_map_batch` and `test_map_every` stay keys of `dobbyengine`.
  Retiring them, with `test_map_upkeep` and the migration of `test_map_every: 0`, belongs to `testmap-upkeep-dobby`.
  Only the `dobbyengine` comments and template text change here, to describe the new refresh.

## To check by hand

- No measurement was run against a real hub.
  Run `lym gate testmap` on the prime after a landing and read the exit record's plan counts and per-package seconds.
- A selector bug is shared by the gate and upkeep.
  The self-check and audit mismatch counts are the only detectors before a landing gate fails, so watch them over the first passes.
- A non-body change to a linked file still rebuilds every map that links it.
  This stays the largest cost, and the plan's miss counts by reason measure it.
