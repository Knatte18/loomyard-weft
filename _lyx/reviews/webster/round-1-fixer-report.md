# Fixer-report — webster round 1

## Changes

- **F1** (`internal/battencli/wire.go`): `wire` resolves `os.Executable` once, returns the error on failure, and `ChildBuildChanged` reads `buildvcs.OfFile` on that captured path.
  Fixed as proposed.
- **F2** (`internal/loomcli`): `spawnDetachedDrive` takes the executable path as a parameter.
  Fixed differently from the proposal in the seam shape: `goDriverDeps` gained an `Executable` field holding the wiring-time path, and `SpawnSuccessor` became `func(executable string) error`, with `handOver` passing `d.deps.Executable`.
  The proposed closure over the `executable` local would have left the path unobservable in `TestGoDriver_Settle`, since that test builds `goDriverDeps` directly and never runs `goDriverDepsFor`.
  The `spawnGoDriver` seam keeps its signature for start and resume, its production closure resolving `os.Executable` and delegating to a new `spawnGoDriverFrom(executable)` seam, which `SpawnSuccessor` calls.
  `TestGoDriver_Settle`'s hand-over rows now assert the spawn call carries `wiredExecutable`.

`lyx gate test ./internal/loomcli ./internal/battencli` is green.

## Disputed

None.

## Deferred

None.
