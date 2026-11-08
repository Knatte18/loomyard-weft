# Decision record — redundant-test-finder

## Goal

A mechanical way to find redundant tests, so they stop piling up, at a cost the gates can carry.
No part fails a gate, refuses a test or deletes one: every finding goes to Code-Review as evidence.

## Scope

In: D1 to D9.

Out:

- Deleting, folding or marking any test.
- Judging `tier2` or `tier3` tests, or tests that may run module code in another process.
- Sharing maps across machines.
- A per-module redundancy test inside the suite.

## Decisions

### D1 — One engine package serves the tool, the map verb and the gate step

`internal/testcover` (new) owns:

- building a package's coverage test binary;
- running each top-level test alone, serially, with its own coverage profile;
- parsing profiles and keying blocks (D3);
- containment and the greedy removable set;
- the coverage map record (D4).

`cmd/testtiming` keeps its flags and report writing and calls the engine instead of its own copies.
The eligibility rules move with the engine unchanged: a test that is skipped, failed or covers nothing is not judged, and neither is one the static spawn scan marks as possibly running module code in another process.
The static spawn scan moves from `cmd/testtiming/spawnscan.go` into `internal/testcover` (new) with the eligibility rules.
The keep scan, which reads each `//testtiming:keep` marker and its reason and is now `scanKeeps` in `cmd/testtiming/keep.go`, moves there with it.
The tool still lists a kept candidate under Kept with its classification unchanged, while D7 step 2 drops a kept test from judging.

Rationale: three callers need the same runner and verdict,
and a `main` package cannot be imported, so the spawn scan and the keep scan leave `cmd/testtiming` with the engine.

### D2 — The tool judges by package-scoped blocks and still shows the module-wide verdict

`go run ./cmd/testtiming -redundancy` keeps one coverage binary per package, instrumented over the whole module as today.
It derives the package view by keeping only blocks in the package's own files.
Candidates and the greedy removable set come from the package view.
The report shows, for each candidate, whether it is also contained under the module-wide view; a cut contained under both is the safer one.
`-pkg` narrows the run and splices only its packages' sections, as today.

Rationale: the module-wide view counts fixture-package blocks and leaves almost every test unique, which is why `docs/benchmarks/test-redundancy.md` lists no `internal/fabricengine` candidate.
One run gives both views, because a module-wide profile holds every block of the package's own files.

### D3 — A block is keyed by content, not by position

A block's key is:

- its package-relative file name;
- its enclosing function, spelled with the receiver type for a method, or a fixed name for code outside any function;
- a hash of the block's source text;
- its ordinal among blocks with the same first three parts.

The map (D4) and the gate step (D7) compare blocks by this key.

Rationale: a profile names a block by line and column, and any edit above it shifts those.
A content key matches an unchanged block across commits by reading the files alone.
An edited block gets a new key that no map test covers, so an edit errs towards no finding.

Bound: two blocks with the same text in one function are told apart by ordinal only.
Inserting a block whose text repeats one in the same function shifts the ordinal of every later identical block,
so each judged test covering one of those blocks can get a wrong cover and a false contained verdict.
Review reading each finding against its covering tests bounds that.

### D4 — A coverage map per package, machine-local under the hub's `.lym`

One JSON record per package, at a path `internal/lymdirs` names under the hub's never-tracked `.lym` tree, shared by every worktree of the hub on the machine.
It holds:

- the commit it was built from and the time it was built;
- a content hash of each Go file the package directory tracks at HEAD, the set D5 compares and its dirty check covers;
- the block key table (D3);
- for each judged top-level test of `tier0` or `tier1`: its tier, a hash of its function's source text, its standalone duration and the indexes of the blocks it covers;
- for a package whose coverage build failed, the failure (D5).

Coverage is scoped to the package itself (`-coverpkg` names only it).
The record is written and read through `internal/state`, never refuses an unknown key, and replaces the old record whole, so a reader sees either the old map or the new one.

Rationale: each machine builds its own map from its own runs; nothing waits on another machine.

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

### D6 — The prime parent's dobby refreshes maps when the prime branch moves

The dobby whose worktree carries the hub's prime parent branch keeps its own record of the prime HEAD its last refresh started from.
On a tick where the prime HEAD differs from that record, or the last refresh reported stale packages the `--max` cap left unbuilt, and no refresh is in flight, it starts `lym gate testmap --max <test_map_batch>`.
The dobby learns of leftovers from D5's exit record alone:
once the child it waited on exits, it reads the record and takes the count only when the record names that child's process ID.
A refresh that was killed, or whose record is missing, unreadable or names another process, counts as reporting none left,
so only a prime HEAD change starts the next one.
Packages a refresh skipped never start another.
`test_map_batch` is a dobby config key with default 5.
A fresh dobby has no record, so its first tick starts a refresh.
The process is started with the binary `selfexec.LymBinary` names, logged, and waited on by the dobby that started it.
It is registered under `runproc.HubOwner` with a new role, `runproc.RoleTestMap` (new), spelled `test-map`.
A refresh is in flight while a live `test-map` record exists, so a restarted dobby reads the record and starts no second refresh.
The prime dobby's stop also calls `runproc.EndRole` for `runproc.HubOwner` with `runproc.RoleTestMap` (new), which ends the child and the processes below it.
A dobby that dies without a stop leaves the child running.
Once the child holds its gate slot it ends on its own within D5's time bound,
and its gate slot and package locks, OS locks, are released when it exits.
A dobby config key, `test_map` in `internal/dobbyengine/template.yaml`, switches the step on or off, default on.
The trigger is independent of the board-tidy landing count.

Rationale: a prime HEAD change is what makes maps stale, whatever landed it.
Bound: a package skipped as locked or changed under a landing waits for the next prime HEAD change.
A modified or untracked Go file in the prime keeps its package skipped until it is committed or removed, and the skip list names the file.
At most one refresh runs per hub, and one a dead dobby left behind lasts, once it holds its gate slot, no longer than D5's time bound.
Only the machine running the prime parent's dobby gets maps; a gate on another machine reports "no map" for each package (D7).

### D7 — An advisory `redundancy` report step in `lym gate test`

A gate key in `gate.yaml` may list `reports: [redundancy]`.
The gate config refuses an unknown report name, naming the known names as the way forward, as it does an unknown lint.
The step runs at the lints' site, after the key's tiers, carried or run, on the worktree's current files, whatever the key's scope.
It is skipped when the worktree's existing report matches by D8's match, taken over the current files in place of HEAD's tree, so a repeat call on an unchanged tree runs no build and no test.
A report that holds an error line, a package with no map or a failed-build map, or a test past the budget never skips the step,
so a later call retries what failed or ran short.
On a hub whose `scoper` is not `go`, the step writes a report saying so and judges nothing; a Go hub left on the `none` scoper therefore gets no judging.

1. It reads the merge base with the parent branch, the base the `card` key uses, through `parentMergeBase` in `internal/gatecli`.
   The gate's existing diff route, `cardChanges` through `impactset.ChangedPaths`, returns only the changed paths.
   For each package holding a changed `_test.go` file, the step lists the package directory's `_test.go` files at the merge base
   through `fabricengine.CodeDirAt` (new) and reads each through `fabricengine.CodeFileAt` (new), both on the warp checkout at a revision;
   a directory absent at the merge base holds no tests.
   `fabricengine.AnchoredFileAt` reads the weft, so it does not serve.
   It looks each current top-level test of a changed `_test.go` file up by package and name across those merge-base files,
   and takes it when that text differs or no such test exists, so a file renamed or moved within its package leaves its unchanged tests untaken.
   A deleted `_test.go` path has no current tests and is passed over.
   A worktree with no parent branch, such as the prime on main, judges nothing, and the report says so.
2. It drops, listing each with its reason: `tier2` and `tier3` tests, tests of a tier the hub does not declare, tests marked `//testtiming:keep` with the reason the marker carries, as D1's keep scan reads them, and tests D1's static spawn scan in `internal/testcover` (new) marks as possibly running module code in another process.
3. It keeps the first `redundancy_max_tests` tests in file then name order, a gate config key with default 40, and lists the rest as not judged.
4. The step joins `internal/gateslot`'s non-tier holders: it takes one gate slot after the key's tiers released theirs and holds it to the step's end;
   the wait for it is not bounded by the step, as D5 states for the verb.
   For each package holding a kept test, it builds a package-scoped coverage binary and runs each such test alone, serially, under the test's tier `env`, building `lym` once when a `tier1` test runs.
   The builds take the slot holder's `HolderEnv`, as in D5.
   Each coverage build and the `lym` build run under `redundancy_build_timeout_min` (D5),
   and a timed-out test is ended with its whole process group and its wait bounded as in D5.
   Each test gets `redundancy_test_timeout_sec` (default 120); a test past it is not judged.
   `redundancy_budget_min` (default 10) counts from the moment the step holds its slot, builds included.
   Once it is spent the step starts no further build or test;
   each test it did not run, the tests of every package it did not build among them, is listed as not judged for the spent budget.
   After the runs, a test that was skipped, failed or covers no block is not judged, with its reason, by D1's eligibility rules.
5. Covers for a judged test are the map tests that still exist with the same function hash, minus the judged test itself, plus the other judged tests.
   The step runs D1's greedy pass, slowest first, over the judged tests with every map test kept, and reports as contained only the tests that pass can drop together.
6. It writes a report into the worktree's gate directory recording the function hash of every test step 1 returned, judged or not,
   and the hash of each Go file of every judged package.
   Per test it gives: contained with its covering tests, largest overlap first; unique; or not judged with the reason.
   A package with no map, or whose map records a build failure, is reported as not judged, the latter with the failure.
   Every judged package's line names its map's commit and age.
   A package whose files differ from its map's hashes is judged,
   and its line also names the files whose text at the merge base differs from the map's hash,
   so the line shows the map's drift from the run's base, not the run's own edits.

When a tier is red the step writes a report saying the gate was red and judges nothing.
The step never changes the gate's result: every error it meets is a line in the report, never an exit code.
Those errors include a build failure, a failing or timed-out test, a missing, unreadable or corrupt map, an unreadable merge base and a failed `lym` build.
A report the step cannot write is logged at Warn, and the gate's result stands.

The shipped template lists the step on the `webster`, `parent-align`, `darn-write` and `code-burler-fix` keys.
It reaches a hub whose base `gate.yaml` tunes those keys, as this hub's `_bolt/_lym/config/gate.yaml` does with no `reports`.
`gates` is no open map, so config load's `yamlengine.FillMissing` call in `internal/configengine/config.go` fills a template key a tuned entry lacks into that entry.
A hub turns the step off on a key by setting `reports: []` on it.

Rationale: a check on new tests alone is what stops redundant tests from piling up.
Judging only the diff's tests against a stored map keeps the cost, once the step holds its slot, to `redundancy_budget_min`:
the touched packages' coverage builds, at most one `lym` build and the tests all count against it.
The build or test running when the budget runs out can add up to the longer of `redundancy_build_timeout_min` and `redundancy_test_timeout_sec`, plus D5's grace either way.
The wait for the step's gate slot lies outside that bound.
A repeat call on a tree its report matches pays only the diff read, the merge-base reads of the test files of each package holding a changed one, and the hashes.
Bound: the step can let a redundant test through, since a changed block counts as uncovered, a test changed since the map is not counted as a cover, and tests past the cap or budget are not judged.
It can flag a test falsely: a map test counts as a cover whenever its own function text is unchanged.
Any change outside that function text, in the diff or landed since the map was built, can stop that test reaching a block it reached when the map was built,
such as a change to a test helper, a non-test callee, a package-level test table, `TestMain` or `testdata`;
a new test covering that block is then reported contained.
Only review reading each finding against the covering tests bounds that.
A new test marked `//testtiming:keep` is listed as not judged with its keep reason, so Code-Review reads the reason in the diff;
that reading is the escape's only check.

### D8 — Code-Burler-Review reads the report when it matches the reviewed tree

The Code-Burler-Review row's stencil gets a Go-filled marker.
Go fills it with the redundancy report's findings when the reviewed HEAD's tree matches the report:
every judged package's Go files hash the same, and step 1 of D7 run on HEAD gives the recorded tests with the recorded function hashes.
A report written on a red tier records no hashes and never matches.
Otherwise it fills a line saying no current report exists and why, naming the red gate when the report is a red one.
`contracts/stencils/loom/loom-rubric-code-review.md` treats each contained test as a test-economy candidate.
The reviewer checks the test's assertions against the covering tests.
It then files a finding to fold or delete the test, naming the covering tests, or accepts it when it pins what coverage cannot see.
A test it accepts gets a `//testtiming:keep <reason>` finding.

Rationale: block coverage does not see assertions (archive tags, deletes on origin, user content in the study), so the reviewer decides.
A content match keeps the report valid across commits of run records and of code outside the judged packages.
Bound: the match ignores the map, so a map rebuilt since the report can make a contained verdict stale; review reading each finding bounds it.

### D9 — `PATTERN-test-speed` names the map, the verb and the step

The background's "Tests run through the gate" section adds `lym gate testmap` and the gate's `redundancy` step as runs that use a tier's `env` and a gate slot.
Its "Test economy" section states the package-scoped verdict with the module-wide column (D2), the map (D4), the report step and its bounds (D7), and that a gate finding is evidence for Code-Review exactly as a report candidate is.

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

None.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - d7-step4-build-and-slot-wait-unbounded (class design, severity MEDIUM, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: the same block sets fed to the engine's verdict give the same contained set and removable set whichever caller feeds them.
- D1: a skipped, failed, coverage-free or out-of-process test is never judged, as in today's report.
- D2: given profiles where a test's unique blocks lie only in another package, the tool lists it as a candidate and marks it not contained module-wide.
- D2: a `-pkg` run rewrites only its packages' sections of an existing report.
- D3: a block whose text is unchanged but shifted by lines above it keeps its key; an edited block gets a new key; same-named methods on two receivers get different keys.
- D4: a map written and read back yields the same tests, hashes and block sets; an unknown key in the record is kept, not refused.
- D5: `lym gate testmap` with no arguments rebuilds only packages with no map, a changed set or content of Go files tracked at HEAD, or a map past the age limit, at most `--max` of them; packages skipped as dirty or locked do not count against `--max`.
- D5: it skips a package with a modified or untracked Go file and one whose lock is held, lists each skipped package with its cause apart from the count the cap left unbuilt, and refuses outside the prime parent's worktree with its way forward.
- D5: as it exits, the verb writes its exit record holding its own process ID and the count of stale packages the cap left unbuilt, zero included.
- D5: a change to a package's own file hashes between its start and the end of its tests discards that package's map, counts against `--max` and reports the package as skipped.
  A HEAD move that leaves them unchanged keeps the map, which records the HEAD read before the package.
- D5: a test past `redundancy_test_timeout_sec` is left out of the map, and the package's other tests are still mapped.
- D5: a package with no test file and one whose coverage build fails each get a map with no tests, the latter naming the failure,
  and the call's exit lists the failure.
  Neither is rebuilt on the next call at the same HEAD while its files are unchanged; the failed one is rebuilt after a HEAD move.
- D6: the dobby starts a refresh when the prime HEAD differs from its record, when D5's exit record names the refresh it waited on and a count of stale packages the cap left unbuilt above zero, or on its first tick.
  It starts none when the prime HEAD is unchanged and the last refresh only skipped packages, none while a live `test-map` record exists, a restarted dobby included, and none when `test_map` is off.
  At an unchanged prime HEAD it starts none for a refresh that was killed or whose exit record is missing or names another process.
- D6: a dobby stop ends a running refresh through `runproc.EndRole` with `runproc.RoleTestMap` (new).
- D7: a new test whose blocks are all covered by unchanged map tests is reported contained, naming those tests.
- D7: a test covering a block of an edited function that no map test and no other judged test covers is reported unique.
- D7: a judged test is never its own cover, and a covering map test whose function text changed since the map is not counted.
- D7: two judged tests covering only each other's blocks are never both reported contained.
- D7: a changed test is found by its function text at the merge base, read through `fabricengine.CodeFileAt` (new); a worktree with no parent branch judges nothing and its report says so.
  A test file renamed or moved within its package leaves its unchanged tests untaken, and a deleted test file adds none.
- D7: a `//testtiming:keep` test with its keep reason, a `tier2` test, a test of an undeclared tier, tests past `redundancy_max_tests`, a timed-out test, a test left unrun once `redundancy_budget_min` is spent, and a test that was skipped, failed or covered no block in the step's run are listed as not judged with their reasons.
  Once the budget is spent no further coverage build starts.
- D7: a package without a map, or whose map records a build failure, is reported as not judged; every judged package's line names its map commit and age,
  and one whose file hashes differ is judged and its line also names the files whose text at the merge base differs from the map.
- D7: a build failure, a failing or timed-out test, an unreadable or corrupt map, an unreadable merge base or a failed `lym` build in the step is a report line and leaves the gate's result as its tiers decided it.
  A failed report write is logged and leaves the result too, and a red tier gives a report that judges nothing.
- D7: a second call on a tree its report matches runs no build and no test, unless the report holds an error line, a package with no map or a failed-build map, or a test past the budget,
  including when the diff also changes a `tier2` test.
- D7: an unknown report name in a gate key is refused with the known names.
- D7: a hub base `gate.yaml` that sets `webster` without `reports` loads with the template's `reports: [redundancy]` on it, and one that sets `reports: []` loads with none.
- D7: on a hub whose `scoper` is not `go`, the step judges nothing and its report says so.
- D8: the marker holds the findings when HEAD's judged packages and changed tests match the report,
  including when the diff also changes a `tier2` test and after a commit touching only `_lym/`.
  It holds the no-report line otherwise, naming the red gate for a report written on a red tier.
- Review: D8's rubric text tells the reviewer to check assertions before filing a fold, delete or keep finding.
- Review: D9's background text states the package-scoped verdict, the map, the verb, the step and their bounds.

## PATTERN changes

- `PATTERN-test-speed` — changes: Writing or running a test: via `lym gate test` by gate key; one `tiergate.Require` tier, lowest that fits; it pins behavior no other test covers, which the gate's redundancy report checks; skips, clocks, `t.Parallel`, `//lym:seam` and `//lym:watch` per the background.

## Notes for the plan writer

- The study's scripts in `docs/research/speed/fabric-test-redundancy-2026-11-08/` show the runner shape the engine reproduces: one binary per package, each test run with `-test.run` anchored on its name and its own coverprofile.
- The gate key `lints` list in `internal/gateslot/template.yaml` is the model for the `reports` list.
- The green record in `internal/gatecli/green.go` shows how the gate already records which tree a result belongs to.
