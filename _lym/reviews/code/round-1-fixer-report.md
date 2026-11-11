```yaml
round: 1
fixed: [F1, F2, F3, F4, F5, F6, F7, F8]
gate: lym gate test --tier 0 --tier 1 ./internal/gatecli, exit 0 for both tiers
```

## Changes

- **F1** — `mapPackage` returns `(bool, error)`; the first result is true only for the before-build linked-file skip.
  `mapPackages` undoes `built++` for it.
  The after-tests and changed-under-tests skips still count.
  The `testMapCmd` Long text, the `mapPackages` and `mapPackage` doc comments and `doc.go` name the new rule.
  A subtest of `TestGateTestMapLinkedPackages` runs with `--max 1` over two packages skipped before their build and a third that builds, which fails when a skip counts.
- **F2** — A stand-in test suffix `LandsLinked`, driven by `testMapLandEnv`, commits a change to a linked package's file while the tests run.
  A new subtest asserts the skip cause `files changed under its tests: linked/linked.go` and that the earlier map is left.
- **F3** — `TestGateTestMap` asserts that `broken`'s failed-build map carries a `Format` with `MapFormatVersion`, the map's `Commit` and `BuiltAt`, and no `Funcs`.
- **F4** — The fixture package `linked` gains a nested module holding only go.mod, a `testdata` Go file, an underscore directory's Go file and a dot directory's Go file.
  The exact `Linked` hashes assert the first is excluded and the other three owned.
- **F5** — A fixture package `follower` links `linked` and sorts after `dirtier`.
  The subtest that dirties a linked file under `dirtier`'s tests names both: `follower` must skip with the before-build cause, which only a fresh status read gives.
- **F6** — Stand-in switches make `go list` and `go env` fail.
  New subtests assert a `go list` failure writes a failed map carrying the marker, and a `go env` failure ends the walk with exit 1 and leaves the earlier maps.
  A package with a syntax error in a non-test file makes `ScanSpawns` fail while `ScanTestFuncs` passes; its subtest asserts the failed map and marker.
  A tier1 test calling a local function value asserts the recorded `Spawn.Unresolved`.
- **F7** — The fixture gains a go.sum, and the first subtest asserts `GoSumHash` and non-empty digests.
  A subtest asserts `TiersDigest` is unchanged by the tiers' `parallel` count and differs after a command change.
  `HubEnvDigest` is pinned by a subtest that writes an env.yaml variable and asserts the digest changes.
- **F8** — The `nestedPackageDirs` doc comment states the build-constraint bound.

Commits: one for F1, one for F8, one for F2 to F7, and one for the self-check changes.
F2 to F7 share one test file, so they are one commit rather than one each.

## Gate repair

The gate refused five comment lines the round added: three over-long lines and two wrapped breaks.
They are in `testmap_integration_test.go` and `testmappackage.go`.
Each is now split at sentence boundaries, one sentence per line, with no change to the code.

## Fixed differently

- **F7**, the `HubEnvDigest` part: the review proposed asserting it non-empty.
  That passes for an empty variable set too, since the hash of an empty string is non-empty, so a subtest instead sets a hub variable and asserts the digest changes.

## Disputed

None.

## Deferred

None.
The review's ScanSpawns failure row was done through a non-test syntax error, which the fixture causes cheaply.

## Rulings

None reached this round.

## Self-check

- fix-consistency, LOW: stale test doc comment naming `caller` and `dirtier` as the linkers. Fixed as proposed.
- fix-consistency, LOW: "such a file" in the `mapPackages` doc read as a Go file. Fixed as proposed.
- fix-code-correctness, LOW: the `HubEnvDigest` non-empty check cannot fail. Fixed differently, as above.
- fix-proposal, LOW: the same `HubEnvDigest` finding, merged with the one above.
- fix-proposal, LOW: a before-build skip still costs a slot hold and a `go list`, which the docs did not say. Fixed as proposed: `doc.go` now says so.
- fix-affected-set: no findings.
  It noted that the `ScanTestFuncs` and `buildLym` failure branches have no row, which the review did not name; left as they were.
