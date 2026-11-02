# fabricengine tier1 hits the 10m go test timeout under load

`fabricengine-tier1-timeout-load` · Task · bug, gate, fabric

## Seen (orch, 2026-11-02, from the darn writer of `tmuxkit-short-socket-path`)

With the load average about 30 and two or three tier1 runs sharing the machine, `internal/fabricengine` tier1 fails with `panic: test timed out after 10m`.
The same timeout hit `beam-attach-no-orch` (gate log `20261102T121452Z-tier1-3498956`) and `explore-subagent-model` (`20261102T122043Z-tier1-3680568`) before their branches had edits, so the cause is not the change under test.
The package takes about 349 s alone on this machine (`test-time-audit-3`), so 10m leaves little room for load.

## Idea

The gate passes no `-timeout`, so go test runs its default 10m.
Set a `-timeout` for tier1 in the gate (config, not a literal), or cut the package time with the `/dev/shm` path of task `tmuxkit-short-socket-path`.
The writer decides after that task has measured the package under `/dev/shm`.
