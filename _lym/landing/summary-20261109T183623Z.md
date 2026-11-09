# lym gate testmap rebuilds stale coverage maps in the prime worktree

`lym gate testmap [--max <n>] [<package>...]` is new.
It fills the per-package coverage maps that `internal/testcover` declared but nothing wrote yet.

## What changes

- `lym gate testmap` rebuilds the map of each named package, or of every stale package when none is named, at most `--max` (default 5) per call, in package path order.
  - A map is stale when it is missing or unreadable;
    when the Go files its directory tracks at HEAD differ from the map's, as a set or in content;
    when it is older than `test_map_max_age_days`;
    or when it records a build failure and HEAD differs from its commit.
    So a failed build is retried once HEAD moves, and never twice at one HEAD.
  - It runs only in the hub's prime worktree. Anywhere else it refuses, naming the prime worktree as the way forward.
  - A package with a modified or untracked Go file is skipped, naming the file.
    So is a package whose build lock another call holds.
    Neither counts against `--max`, and both are skipped before any build.
  - For each package it reads HEAD and the package's file hashes first, and the map records that HEAD.
    It reads them again once the tests end.
    When a hash differs, as when a landing changed the package's files, it discards the map and reports the package skipped; that package counts against `--max`.
    A HEAD move that leaves the package's files alone keeps the map.
  - Its tests are the package's top-level Test functions whose `tiergate.Require` names tier0 or tier1 and whose tier the hub declares.
    A package with none gets a map of its file hashes and no tests, with no build.
    Otherwise the call takes one gate slot and holds it to its end.
    It builds the package's coverage binary under `gateslot.HolderEnv`, then runs each test alone in the package directory under its tier's `env`.
  - Each build is bounded by `redundancy_build_timeout_min` and each test by `redundancy_test_timeout_sec`.
    Past the bound the whole process group is killed, and the wait for its exit and output lasts at most the gate's ten-second pipe wait.
    A failed or timed-out build gives a map with no tests that names the failure.
    A timed-out test is listed and left out, and the package's other tests are still mapped.
  - It prints `built`, `skipped` with causes, `failed` with failures, `timed_out` and `unbuilt`, the count of stale packages the cap left unbuilt.
    As it exits it writes `testcover.ExitRecord`, holding its pid and that count, zero included.
- `internal/gateslot` gains the keys `test_map_max_age_days` (7), `redundancy_build_timeout_min` (10) and `redundancy_test_timeout_sec` (120).
  All three are cross-pair, and all three are refused below 1.
- `internal/fabricengine` gains `Fabric.CodeDirAt` and `Fabric.CodeFileAt`, which read the code checkout's files as stored in a commit.
- `internal/hubgeom` gains `IsPrime`, a spawn-free read of the `.git` entry, and `PrimeWorktreePath` for the refusal's way forward.
- `internal/testcover`:
  - `Toolchain` gains `BuildTimeout`, `TestTimeout` and `Grace`, and `RunResult` gains `TimedOut`;
  - `ScanTestFuncs` returns each top-level Test function with its tier and the hash of its source text;
  - `Dir` names the hub's maps directory;
  - `ExitRecord`, `WriteExitRecord` and `ReadExitRecord` are the refresh's exit record.

  `lymdirs.TestMapExitFileName` names the record's file.
- `lym gate test`'s environment set-up moves into `gateEnv`, which the new verb shares.
  The `lym` prebuild now kills its whole process group when its context is cancelled.
- The `internal/gatecli`, `internal/gateslot` and `internal/testcover` package docs and `docs/overview.md` describe the verb and the keys.

## Design decisions

- `CodeDirAt` and `CodeFileAt` are methods on `Fabric`, not package functions.
  One call reads every package through one cached repository, instead of opening it for every file.
  They take a commit SHA, the one `Fabric.HeadSHA` returns.
  `internal/gitrepo`'s existing reads were enough, so it is unchanged.
- `IsPrime` and `PrimeWorktreePath` sit in `internal/hubgeom`, which the decision record names for the prime check.
  The package was not in the record's scope list.
- Packages are found by walking the module the way `go list ./...` does, leaving out testdata, dot and underscore directories and nested modules.
  So choosing packages spawns no `go`.
  The module root is the `dir` of the lowest declared of tier0 and tier1.
- The `lym` build is handed to tests whose tier sets `prebuilt_lym`, as a tier run does, rather than to tier1 by number.
  It is built at most once per call, under the build timeout, into the pool's prebuilt store.
- The build lock is checked before the cap, so a package another call is building is reported skipped and never counted as unbuilt.
- The coverage binary covers the package alone, since a map keeps only the package's own blocks.
  A test that failed, skipped or covered none of the package's blocks stays out of the map.
- Only a call that gets past the prime check writes an exit record, because the record belongs to the prime's refresh.
- `PATTERN-test-speed` is unchanged.
  The entry `review-reads-redundancy` names the map, the verb and the step there.

## Tests

No test launches a real test binary or `go test`.
`internal/gatecli`'s `TestGateTestMap` re-executes the test binary as the stand-in `go` and as the coverage binary it "builds".
It runs calls in order over one hub, covering:

- the refusal outside the prime and the `--max 0` refusal;
- the cap and its unbuilt count;
- the dirty and locked skips, which do not count;
- a failed build kept at one HEAD and retried after a HEAD move;
- a timed-out test left out while the package's other tests are mapped;
- a package whose files change under its tests, discarded;
- a HEAD move outside a package, which keeps the map with the earlier commit;
- the age limit, a named fresh package, and the exit record.

`internal/testcover` adds a timed-out run row, the test function scan and the exit record round trip.
`internal/fabricengine` covers the two commit reads against an edited and an untracked file.
`internal/gateslot` adds the three keys' defaults and refusals.

## To check by hand

A real `lym gate testmap --max 1` in the prime worktree would show a map built from a live package under `<hub>/_bolt/.lym/testcover/`.
