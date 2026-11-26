# Fixing the darn-write comment lint failure

`lym gate test --gate darn-write` stayed red twice after the lint fix:
the memory guard killed the `cmd/lym` tier1 test binary while `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` ran.
The user slice had under 2 GB of its 16384 MB left each time, and the test files this branch changes are all in `internal/loomcli`.
The red tier also stopped the reach, redundancy and mutation reports.
