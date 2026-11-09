# lym gate testmap rebuilds stale coverage maps in the prime parent's worktree

`testmap-verb` · Task · feature, testkit, gate · running · Drift-Expect

Decision record: [Sketch](decision-testmap-verb.md)

Depends on: [`testcover-engine`](design-testcover-engine.md)

Filed from the Sketch `redundant-test-finder`.

Takes: testcover-engine's runner, block keys and map record
Delivers: the `lym gate testmap [--max <n>] [<package>...]` cobra verb with staleness, dirty and lock skips, mid-package hash check, timeouts with process-group kill and grace, failed-build maps and the exit record; fabricengine.CodeDirAt and CodeFileAt on the warp checkout; gate config keys test_map_max_age_days, redundancy_build_timeout_min and redundancy_test_timeout_sec
Exports: fabricengine.CodeDirAt, fabricengine.CodeFileAt, the exit record type and its read, and the per-tier run-alone with timeout that the gate step reuses

The part changes these packages:

- internal/gatecli (about 600 changed lines)
- internal/gateslot (about 80 changed lines)
- internal/fabricengine (about 80 changed lines)
- internal/gitrepo (about 40 changed lines)
- internal/lymdirs (about 10 changed lines)
- internal/testcover (about 120 changed lines)
