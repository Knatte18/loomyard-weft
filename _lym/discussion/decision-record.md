## Goal

lym gate testmap rebuilds stale coverage maps in the prime parent's worktree

Takes: testcover-engine's runner, block keys and map record
Delivers: the `lym gate testmap [--max <n>] [<package>...]` cobra verb with staleness, dirty and lock skips, mid-package hash check, timeouts with process-group kill and grace, failed-build maps and the exit record; fabricengine.CodeDirAt and CodeFileAt on the warp checkout; gate config keys test_map_max_age_days, redundancy_build_timeout_min and redundancy_test_timeout_sec
Exports: fabricengine.CodeDirAt, fabricengine.CodeFileAt, the exit record type and its read, and the per-tier run-alone with timeout that the gate step reuses

## Scope

The part changes these packages:

- internal/gatecli (about 600 changed lines)
- internal/gateslot (about 80 changed lines)
- internal/fabricengine (about 80 changed lines)
- internal/gitrepo (about 40 changed lines)
- internal/lymdirs (about 10 changed lines)
- internal/testcover (about 120 changed lines)

## Decisions

### D5 — `lym gate testmap` rebuilds stale maps, only in the prime parent's worktree

`lym gate testmap [--max <n>] [<package>...]` (new) rebuilds the map of each named package, or of every stale package when none is named, at most `--max` packages per call, in package path order.
A package is stale when:

- it has no map;
- the Go files its directory tracks at HEAD, as a set or in the content of one of them, differ from what its map recorded;
- or its map is older than `test_map_max_age_days`, a gate config key with default 7.

The verb runs only in the worktree carrying the hub's prime parent branch, read through `internal/hubgeom`.
In any other worktree it refuses, naming that worktree as the way forward.
It skips a package whose directory holds a modified or untracked Go file, since the map records HEAD as its commit;
an untracked Go file therefore makes a package skipped, never stale.
It reads the Go files a directory tracks at HEAD through `fabricengine.CodeDirAt` (new) and `fabricengine.CodeFileAt` (new) at the revision HEAD, as D7 step 1 does,
and the modified or untracked files through the fabric status read, `Fabric.Status`;
no git beyond the HEAD SHA read, which `PATTERN-fabric-boundary` exempts, runs outside `internal/fabricengine`.
It holds one gate slot for the whole call and a per-package lock taken through `internal/lock`; a package another call holds is skipped.
A package skipped as dirty or locked is skipped before any build and does not count against `--max`.
Before each package it records HEAD and the package's file hashes, and after the package's tests it reads the hashes again.
The map records the HEAD read before the package as its commit.
When a hash differs, as when a Finalize landing changes the package's files mid-package, it discards that package's map and reports the package as skipped;
such a package counts against `--max`.
A landing that leaves the package's files unchanged keeps the map, as any other dependency change does.
Each test runs alone in its own process group under its tier's `env` from `gate.yaml` and gets `redundancy_test_timeout_sec` (D7);
the builds take the slot holder's `HolderEnv` from `internal/gateslot`.
A timed-out test is ended with its whole process group,
and the verb then waits for the test's exit and output at most a fixed grace, so a descendant holding the output pipe cannot stall it.
A timed-out test is left out of the map, and so is a test whose tier the hub does not declare.
When any `tier1` test runs, the verb builds `lym` once per call and hands it on as a tier run does.
Each coverage build and the `lym` build run under `redundancy_build_timeout_min`, a gate config key with default 10;
a build past it is ended with its process group like a test and counts as failed.
A package whose coverage build fails, or which has no test file or no `tier0` or `tier1` test, gets a map recording its Go file hashes and no tests,
with the build failure as its cause where one occurred, and it counts against `--max`.
Such a map is stale again when its files change or it passes `test_map_max_age_days`;
one recording a build failure is also stale once HEAD differs from its recorded commit,
so a failure caused outside the package is retried after the next prime HEAD move, never twice at one HEAD.
Each spawned process is logged through `internal/logger` and waited on.
It exits listing each skipped package with its cause, the modified or untracked file, the held lock or the files changed under its tests,
and each package whose build failed with the failure, apart from the count of stale packages the `--max` cap left unbuilt.
That count reaches its caller through an exit record the verb writes as it exits:
a record written through `internal/state` at a path `internal/lymdirs` names under the hub's `.lym`, holding the verb's process ID and the count.

Bound: it writes nothing but map records and its exit record, runs only the package's own `tier0` and `tier1` tests, and holds one slot for at most `--max` packages per call.
In time, the wait for its gate slot is not bounded by the verb.
Once it holds the slot, a call lasts at most one `lym` build plus, per package it builds, one coverage build,
each under `redundancy_build_timeout_min`, and its tests' count times `redundancy_test_timeout_sec` plus the grace.
A change to a dependency or to a non-Go file of the package directory, such as testdata or an embedded file, does not mark a map stale;
the age limit bounds how long such drift lasts.

## Constraints

- Per-test coverage needs each test in its own process run: `runtime/coverage` counters are process-wide, so tests under `t.Parallel` mix their counts.
  The map cannot come from a normal tier run.
- No test of this feature launches a test binary or `go test`: the runner's tests use a stand-in `go` and fake profiles, as `cmd/testtiming`'s prebuild test does.
- `PATTERN-gate-slot`: the verb and the step run tests only while holding gate slots.
- `PATTERN-fabric-boundary`: the verb's tracked-file and status reads (D5) and the step's merge-base reads (D7 step 1) go through `internal/fabricengine`.
- `PATTERN-config-layers`: every new key has a default and loads through `configengine`; no meaning of an existing key changes, so no reinterpretation ships.
- `PATTERN-cli-cobra`: `lym gate testmap` joins the existing `lym gate` cobra subtree.
- `PATTERN-refusal-way-forward`: the verb's refusal outside the prime worktree and the unknown report name refusal each name their way forward, and a test reaches each.
- `PATTERN-code-conventions`: the step's keys come from `gate.yaml`, never a hard-coded key list.
- The prime parent's worktree is updated in place when a run lands, so its HEAD tracks the prime branch.
  `internal/fabricengine/merge.go`'s merge runs `MergeStart` in the parent's warp checkout under the weft write lock.

## Auto-mode assumptions

- The full design is built, not only the tool change: the goal asks that redundant tests stop piling up, which needs the check on new tests.

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - build-timeout-grace-untested (class scope, severity MEDIUM, open in rounds 1)
  - coverage-build-cwd-not-module-root (class design, severity MEDIUM, open in rounds 1)
  - signal-recorded-as-build-failure (class design, severity MEDIUM, open in rounds 1)
  - staleness-file-set-content-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D5: `lym gate testmap` with no arguments rebuilds only packages with no map, a changed set or content of Go files tracked at HEAD, or a map past the age limit, at most `--max` of them; packages skipped as dirty or locked do not count against `--max`.
- D5: it skips a package with a modified or untracked Go file and one whose lock is held, lists each skipped package with its cause apart from the count the cap left unbuilt, and refuses outside the prime parent's worktree with its way forward.
- D5: as it exits, the verb writes its exit record holding its own process ID and the count of stale packages the cap left unbuilt, zero included.
- D5: a change to a package's own file hashes between its start and the end of its tests discards that package's map, counts against `--max` and reports the package as skipped.
  A HEAD move that leaves them unchanged keeps the map, which records the HEAD read before the package.
- D5: a test past `redundancy_test_timeout_sec` is left out of the map, and the package's other tests are still mapped.
- D5: a package with no test file and one whose coverage build fails each get a map with no tests, the latter naming the failure,
  and the call's exit lists the failure.
  Neither is rebuilt on the next call at the same HEAD while its files are unchanged; the failed one is rebuilt after a HEAD move.
