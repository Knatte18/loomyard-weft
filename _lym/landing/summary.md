# Add `lym gate testmap` to rebuild stale coverage maps

`lym gate testmap [--max <n>] [<package>...]` rebuilds the per-test coverage map of each named package, or of every stale package when none is named.
The maps feed the redundant-test check, so they have to be refreshed without a normal tier run:
`runtime/coverage` counters are process-wide, and tests under `t.Parallel` would mix their counts.
Each test therefore runs alone in its own process group.

## What changes

- New verb `lym gate testmap` in the `lym gate` cobra subtree (`internal/gatecli`).
  It builds at most `--max` packages per call, in package path order.
- A package is stale when it has no map, when the Go files its directory tracks at HEAD differ from the map's record (as a set or in the content of one file), or when its map is older than `test_map_max_age_days`.
  A change to a dependency or to a non-Go file of the directory does not mark a map stale; the age limit bounds that drift.
- The verb runs only in the worktree carrying the hub's prime parent branch (read through `internal/hubgeom`).
  Elsewhere it refuses and names that worktree as the way forward.
- A package whose directory holds a modified or untracked Go file is skipped, since the map records HEAD as its commit.
  A package whose lock (`internal/lock`) another call holds is skipped too.
  Neither skip counts against `--max`.
- The verb holds one gate slot for the whole call.
  Before each package it records HEAD and the package's file hashes, and it reads the hashes again after the package's tests.
  When a hash differs, as when a landing changes the package's files mid-package, the map is discarded and the package is reported as skipped; this counts against `--max`.
  A landing that leaves the files unchanged keeps the map, which records the HEAD read before the package.
- Each test runs under its tier's `env` from `gate.yaml` and gets `redundancy_test_timeout_sec`.
  A timed-out test is ended with its whole process group, and the verb then waits for its exit and output for a fixed grace, so a descendant holding the output pipe cannot stall the call.
  A timed-out test, and a test whose tier the hub does not declare, is left out of the map.
- `lym` is built once per call when any `tier1` test runs.
  That build and every coverage build run under `redundancy_build_timeout_min`; a build past it is ended with its process group and counts as failed.
- A package with no test file or no `tier0` or `tier1` test, and a package whose coverage build fails, get a map that records the Go file hashes and no tests.
  The failed build is recorded as the map's cause.
  Such a map is stale again when its files change or it passes the age limit.
  A map recording a build failure is also stale once HEAD differs from its recorded commit, so a failure is retried after the next prime HEAD move and never twice at one HEAD.
- On exit the verb lists each skipped package with its cause (the modified or untracked file, the held lock, or the files changed under its tests) and each package whose build failed with the failure.
  It writes an exit record holding its process ID and the count of stale packages the `--max` cap left unbuilt, zero included.
  The record goes through `internal/state` to a path `internal/lymdirs` names under the hub's `.lym`.
  `internal/testcover` owns the record type and its read.
- New gate config keys, each with a default and loaded through `configengine`: `test_map_max_age_days` (7), `redundancy_build_timeout_min` (10) and `redundancy_test_timeout_sec`.
  No existing key changes meaning.
  `internal/gateslot` carries the keys and its `template.yaml`.

## Design decisions

- `fabricengine.CodeDirAt` and `fabricengine.CodeFileAt` read the Go files a directory tracks at a revision from the warp checkout.
  Status reads go through `Fabric.Status`.
  Beyond the HEAD SHA read, no git runs outside `internal/fabricengine` (`PATTERN-fabric-boundary`).
- The per-tier run-alone with timeout lives in `internal/testcover` (`runner.go`, `testfuncs.go`) so the later gate step can reuse it.
  The exit record type and its read are exported for the same reason.
- Tests are run only while holding a gate slot (`PATTERN-gate-slot`).
  The builds take the slot holder's `HolderEnv` from `internal/gateslot`.
- The wait for the gate slot is not bounded by the verb.
  Once it holds the slot, a call lasts at most one `lym` build plus, per package it builds, one coverage build, and the tests' count times `redundancy_test_timeout_sec` plus the grace.
- No test of this feature launches a test binary or `go test`; the runner tests use a stand-in `go` and fake profiles.

## To check by hand

- Run `lym gate testmap` in the prime parent's worktree against a real hub, and confirm that a real coverage build, the `lym` build and a real timed-out test behave as the stand-in tests do.
  The stand-in tests never launch a real test binary.
- Coverage build working directory: a coverage build must run in the module root (`internal/gatecli/testmap.go`, `internal/gatecli/testmappackage.go`).
  Confirm the run-in-module-root fix holds for a package in a nested module.
- Signals versus build failures: a signal that ends a build must stay out of the failure map and must not be recorded as a build failure (`internal/gatecli/testmappackage.go`).
  Confirm a killed build is reported as skipped and not recorded as a failed-build map.
- Build timeout and grace: confirm the test for a build that outlives `redundancy_build_timeout_min` and its grace covers the kill path (`internal/gatecli/testmap_integration_test.go`).
- Staleness by file set and by content: confirm the tests cover both a changed set and a changed content of the Go files tracked at HEAD (`internal/gatecli/testmap_integration_test.go`).
