# Webster round 1 — fixer report

Both findings were checked against the code and the plan and found valid.
Both are fixed, each in its own commit, with `lyx gate test` and `lyx gate test --tags integration` green on the touched packages and `lyx loom lint-comments` clean.

## Fixed

### F1 — `LYX_LOG_LEVEL=debug` route to a daemon Debug trace (commit `869feb6e9`)

Confirmed: `cmd/lyx/main.go`'s root `PersistentPreRunE` called `logger.SetVerbosity(verbosity)` unconditionally, and with no `-v` that reset the init-time `LYX_LOG_LEVEL` level to Warn.
I fixed this where the reviewer said to make the claim true.
The root hook now calls `SetVerbosity` only when `-v` is given, so a spawned `lyx reed watchdog` keeps the `LYX_LOG_LEVEL=debug` it inherits.
An explicit `-v`/`-vv` still wins over the variable, as the logger's package doc states.

- `cmd/lyx/main.go`: `SetVerbosity` is now called only when `verbosity > 0`.
- `internal/logger/logger.go`: the `SetVerbosity` doc now says the root hook calls it when the flag is given.
- `cmd/lyx/main_test.go`: new `TestRootHook_NoVerboseFlagKeepsTheLevelSetAtInit` pins that a Debug record still reaches stderr after the hook runs with no `-v`. This behavior had no test. It is not parallel because it changes global logger state.
- `cmd/lyx/testdata/test-budget.yaml`: the `cmd/lyx` row goes from 72 to 73 for that test.
- `docs/benchmarks/daemon-idle-cpu.md`: the deviation entry stays as a record of that run, with one added line noting the root hook no longer resets the level.

The help text, `reedcli/doc.go`, the `SpawnWatchdog` doc and the `Debug` doc now describe working behavior, so they are unchanged.

### F2 — boot refusals at the planned site reached by no test (commit `40cd76cf5`)

Confirmed: for `Up`, the refusals fired in `withBootOpLockKeepingZoom`, so the call in `ensureServerAndSessionLocked` was never exercised.
I kept both call sites on purpose.
The bracket's zoom read is a tmux round trip, so without its own copy of the checks `Up` and `Resume` would reach tmux before refusing.
`validateBootConfig` already runs at both sites in the same way.

- `internal/reedengine/lifecycle_test.go`: in `TestUp_BootValidation`, each refusal row now also calls `ensureServerAndSessionLocked()` directly. It asserts the same refusal text and zero tmux calls across both calls. This adds assertions to existing rows and no new top-level test, so the `internal/reedengine` budget is unchanged.
- `internal/reedengine/zoom.go`: the `withBootOpLockKeepingZoom` doc now explains that the bracket repeats the boot's refusals because its zoom read would otherwise reach tmux first.

## Disputed

None.

## Deferred

None.
