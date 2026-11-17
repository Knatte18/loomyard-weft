```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. For each function and branch the diff changes, I ran one concrete case through the code.

**Cases run**

- **`sheddriver.handOver`, tree with uncommitted producer output and the build changed** (`internal/sheddriver/drive.go:354-385`).
  It holds any pause, reads the on-disk build, appends the `SettleHandOver` record, hands back with the hand-over rule (report and park), sets `ExitHandOver`, releases the lock and spawns the successor.
  Correct: nothing reads the tree now, so a dirty tree no longer diverts this path.
- **`handOver`, `holdPause` fails.** It returns the error before any record is written.
  Correct and unchanged.
- **`handOver`, `OnDiskBuild` fails.** `handBackAs` hands back with "reading the on-disk build" and writes no hand-over record.
  Correct and unchanged.
- **`handOver`, `AppendSettleRecord` fails.** It hands back with the error and spawns nothing.
  Correct and unchanged.
- **`handOver`, `ReleaseLock` fails.** It returns a wrapped error after the record and the park are already written.
  Unchanged by the diff, so outside this round.
- **`handOver`, `SpawnSuccessor` fails** (the "no such binary" case). It logs a warning, returns nil and leaves the run parked with its report.
  Correct, and pinned by the existing failing-spawn row.
- **Successor started over the dirty tree.** Nothing in the drive wiring refuses a dirty task tree any more: the only tree read was the removed `DirtyPaths` (grep of `internal/loomcli`).
  The successor re-steps under the new build, so `LastStepBuildChanged` turns false and no second hand-over fires.
  If the successor decides at the same stop before it steps, `handedOver(records, key, onDisk.Label())` is true for that key and build (`drive.go:336-343`, `driverules.go:264`).
  So the hand-over row does not match again, and no hand-over loop is possible.
- **Same stop key, newer build again** (a second deploy): `Build` is a different label, so one more hand-over runs.
  That is the intended rule of one hand-over per stop key and build.
- **`goDriverDepsFor` wiring with the `DirtyPaths` line removed** (`internal/loomcli/drivewiring.go:301-310`).
  The seam literal compiles, and `go build` and `go vet` pass for `./internal/loomcli` and `./internal/sheddriver`.
  `fabricengine`, `filepath`, `loomengine` and `lymcwd` are still used elsewhere in the file, and the `strings` import was correctly dropped from `drive.go`.
- **`dirtyPathsOutsideRecords` removed.** No reference remains.
  `loomengine.PathsOutsideRunRecords` keeps its other production caller at `internal/battencli/wire.go:1000`.
- **`TestGoDriver_Settle` with the `dirty` field and its row removed.** The seam literal no longer sets `DirtyPaths`, and the build passes.

**Findings**

None.

**Tool calls**

1. Grepped `internal/loomcli` and `internal/shedverbs` for tree checks and read `internal/battencli/wire.go:975-1002,1465-1490`.
   My context lacked whether the successor's drive or step path still refuses a dirty tree, and the batten caller of `PathsOutsideRunRecords`.
