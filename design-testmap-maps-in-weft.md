# Keep the coverage maps in each pair's weft branch, committed and pushed by lym gate testmap

`testmap-maps-in-weft` · Task · enhancement, gate, testkit, dobby · draft

Decision record: [Sketch](decision-testmap-maps-in-weft.md)

Filed from the Sketch `testmap-checked-in`.

Takes: the maps under <bolt>/.lym/testcover/, fabricengine's scoped weft commit (CommitRecordsPaths), SpawnStatusPush, ReconcileOrigin, and loomcli's unexported push verdict in prepush.go
Delivers: maps tracked under each worktree's _lym/testcover/, written compact and only on change, committed and pushed by lym gate testmap in the prime or a task, tolerated by preflight and committed before a weft pull, with upkeep off by default and a one-time import of the bolt maps
Exports: testcover.Dir on the worktree's anchor, a scratch accessor for the exit record, a compact form of state's JSON writer, and one exported fabricengine function holding the run-scope push verdict that loomcli's pre-push check and the verb both call

The part changes these packages:

- internal/gatecli (about 900 changed lines)
- internal/fabricengine (about 450 changed lines)
- internal/testcover (about 250 changed lines)
- internal/loomcli (about 200 changed lines)
- internal/state (about 80 changed lines)
- internal/beamcli (about 40 changed lines)
- internal/runproc (about 30 changed lines)
- internal/dobbyengine (about 30 changed lines)
- internal/preflight (about 30 changed lines)
- internal/lymdirs (about 15 changed lines)
- pattern (about 15 changed lines)
