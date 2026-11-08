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
The eligibility rules move with the engine unchanged: a test that is skipped, failed or covers nothing is not judged, and neither is one the static scan in `cmd/testtiming/spawnscan.go` marks as possibly running module code in another process.

Rationale: three callers need the same runner and verdict, and a `main` package cannot be imported.

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

Bound: two blocks with the same text in one function are told apart by ordinal only, so inserting one before the other can match the wrong block; the finding is evidence for review, so that costs at most one wrong hint.

### D4 — A coverage map per package, machine-local under the hub's `.lym`

One JSON record per package, at a path `internal/lymdirs` names under the hub's never-tracked `.lym` tree, shared by every worktree of the hub on the machine.
It holds:

- the commit it was built from and the time it was built;
- a content hash of each Go file in the package directory;
- the block key table (D3);
- for each judged top-level test of `tier0` or `tier1`: its tier, a hash of its function's source text, its standalone duration and the indexes of the blocks it covers.

Coverage is scoped to the package itself (`-coverpkg` names only it).
The record is written and read through `internal/state`, never refuses an unknown key, and replaces the old record whole, so a reader sees either the old map or the new one.

Rationale: each machine builds its own map from its own runs; nothing waits on another machine.

### D5 — `lym gate testmap` rebuilds stale maps, only in the prime parent's worktree

`lym gate testmap [--max <n>] [<package>...]` (new) rebuilds the map of each named package, or of every stale package when none is named, at most `--max` packages per call, in package path order.
A package is stale when:

- it has no map;
- a Go file in its directory, tracked or untracked, differs from the hash its map recorded;
- or its map is older than `test_map_max_age_days`, a gate config key with default 7.

The verb runs only in the worktree carrying the hub's prime parent branch, read through `internal/hubgeom`.
In any other worktree it refuses, naming that worktree as the way forward.
It skips a package with a tracked or untracked Go file that differs from HEAD, since the map records HEAD as its commit, and reports it as skipped.
It holds one gate slot for the whole call and a per-package lock taken through `internal/lock`; a package another call holds is skipped.
Each test runs alone under its tier's `env` from `gate.yaml`; a test whose tier the hub does not declare is left out of the map.
When any `tier1` test runs, the verb builds `lym` once per call and hands it on as a tier run does.
Each spawned process is logged through `internal/logger` and waited on.
It exits reporting how many stale packages remain unbuilt.

Bound: it writes nothing but map records, runs only the package's own `tier0` and `tier1` tests, and holds one slot for at most `--max` packages per call.
A dependency change outside the package directory does not mark a map stale; the age limit bounds how long such drift lasts.

### D6 — The prime parent's dobby refreshes maps when the prime branch moves

The dobby whose worktree carries the hub's prime parent branch keeps its own record of the prime HEAD its last refresh started from.
On a tick where the prime HEAD differs from that record, or the last refresh reported stale packages left unbuilt, and no refresh is in flight, it starts `lym gate testmap --max <test_map_batch>`.
`test_map_batch` is a dobby config key with default 5.
A fresh dobby has no record, so its first tick starts a refresh.
The process is started with the binary `selfexec.LymBinary` names, logged, and waited on by the dobby.
A dobby config key, `test_map` in `internal/dobbyengine/template.yaml`, switches the step on or off, default on.
The trigger is independent of the board-tidy landing count.

Rationale: a prime HEAD change is what makes maps stale, whatever landed it.
Bound: a package skipped as dirty waits for the next prime HEAD change.
Only the machine running the prime parent's dobby gets maps; a gate on another machine reports "no map" for each package (D7).

### D7 — An advisory `redundancy` report step in `lym gate test`

A gate key in `gate.yaml` may list `reports: [redundancy]`.
The gate config refuses an unknown report name, naming the known names as the way forward, as it does an unknown lint.
The step runs at the lints' site, after the key's tiers, carried or run, on the worktree's current files, whatever the key's scope:

1. It takes the top-level tests whose function text differs from, or is absent at, the merge base with the parent branch, the base the `card` key uses, read through the gate's existing diff route.
2. It drops `tier2` and `tier3` tests, tests of a tier the hub does not declare, tests marked `//testtiming:keep`, and tests D1's eligibility rules leave unjudged, listing each with its reason.
3. It keeps the first `redundancy_max_tests` tests in file then name order, a gate config key with default 40, and lists the rest as not judged.
4. For each package holding a kept test, it builds a package-scoped coverage binary and runs each such test alone, serially, under the test's tier `env`, building `lym` once when a `tier1` test runs.
   Each test gets `redundancy_test_timeout_sec` (default 120), and the step stops starting tests after `redundancy_budget_min` (default 10); a test past either is not judged.
5. Covers for a judged test are the map tests that still exist with the same function hash, minus the judged test itself, plus the other judged tests.
   The step runs D1's greedy pass, slowest first, over the judged tests with every map test kept, and reports as contained only the tests that pass can drop together.
6. It writes a report into the worktree's gate directory recording, per judged package, the hash of each Go file, and per judged test its function hash.
   Per test it gives: contained with the fewest covering tests that cover its blocks, unique, or not judged with the reason.
   A package with no map is reported as not judged; a package whose files differ from its map's hashes is judged, and the report names the map commit, its age and the changed files.

When a tier is red the step writes a report saying the gate was red and judges nothing.
The step never changes the gate's result: a build failure, a failing or timed-out test, or a missing map is a line in the report.
The shipped template lists the step on the `webster`, `parent-align`, `darn-write` and `code-burler-fix` keys.

Rationale: a check on new tests alone is what stops redundant tests from piling up.
Judging only the diff's tests against a stored map keeps the cost to one coverage build per touched package, at most one `lym` build, and those tests' own run time, capped by the budget.
Bound: the step can let a redundant test through, since a changed block counts as uncovered, a test changed since the map is not counted as a cover, and tests past the cap or budget are not judged.
It flags a test falsely only when a map test lost coverage since the map was built, which the age limit of D5 bounds; review reads each finding before acting.

### D8 — Code-Burler-Review reads the report when it matches the reviewed tree

The Code-Burler-Review row's stencil gets a Go-filled marker.
Go fills it with the redundancy report's findings when the reviewed HEAD's tree matches the report: every judged package's Go files hash the same, and step 1 of D7 run on HEAD gives the same tests with the same function hashes.
Otherwise it fills a line saying no current report exists and why.
`contracts/stencils/loom/loom-rubric-code-review.md` treats each contained test as a test-economy candidate.
The reviewer checks the test's assertions against the covering tests.
It then files a finding to fold or delete the test, naming the covering tests, or accepts it when it pins what coverage cannot see.
A test it accepts gets a `//testtiming:keep <reason>` finding.

Rationale: block coverage does not see assertions (archive tags, deletes on origin, user content in the study), so the reviewer decides.
A content match keeps the report valid across commits of run records and of code outside the judged packages.

### D9 — `PATTERN-test-speed` names the map, the verb and the step

The background's "Tests run through the gate" section adds `lym gate testmap` and the gate's `redundancy` step as runs that use a tier's `env` and a gate slot.
Its "Test economy" section states the package-scoped verdict with the module-wide column (D2), the map (D4), the report step and its bounds (D7), and that a gate finding is evidence for Code-Review exactly as a report candidate is.

## Constraints

- Per-test coverage needs each test in its own process run: `runtime/coverage` counters are process-wide, so tests under `t.Parallel` mix their counts.
  The map cannot come from a normal tier run.
- No test of this feature launches a test binary or `go test`: the runner's tests use a stand-in `go` and fake profiles, as `cmd/testtiming`'s prebuild test does.
- `PATTERN-gate-slot`: the verb and the step run tests only while holding gate slots.
- `PATTERN-config-layers`: every new key has a default and loads through `configengine`; no meaning of an existing key changes, so no reinterpretation ships.
- `PATTERN-cli-cobra`: `lym gate testmap` joins the existing `lym gate` cobra subtree.
- `PATTERN-refusal-way-forward`: the verb's refusal outside the prime worktree and the unknown report name refusal each name their way forward, and a test reaches each.
- `PATTERN-code-conventions`: the step's keys come from `gate.yaml`, never a hard-coded key list.

## Auto-mode assumptions

- The full design is built, not only the tool change: the goal asks that redundant tests stop piling up, which needs the check on new tests.
- The prime parent's worktree is updated in place when a run lands, so its HEAD tracks the prime branch.

## Open risks

None.

## Acceptance criteria

- D1: the same block sets fed to the engine's verdict give the same contained set and removable set whichever caller feeds them.
- D1: a skipped, failed, coverage-free or out-of-process test is never judged, as in today's report.
- D2: given profiles where a test's unique blocks lie only in another package, the tool lists it as a candidate and marks it not contained module-wide.
- D2: a `-pkg` run rewrites only its packages' sections of an existing report.
- D3: a block whose text is unchanged but shifted by lines above it keeps its key; an edited block gets a new key; same-named methods on two receivers get different keys.
- D4: a map written and read back yields the same tests, hashes and block sets; an unknown key in the record is kept, not refused.
- D5: `lym gate testmap` with no arguments rebuilds only packages with no map, a changed or untracked file, or a map past the age limit, at most `--max` of them.
- D5: it skips a package whose files differ from HEAD, skips one whose lock is held, and refuses outside the prime parent's worktree with its way forward.
- D6: the dobby starts a refresh when the prime HEAD differs from its record, when stale packages were left, or on its first tick, never a second while one is in flight, and none when `test_map` is off.
- D7: a new test whose blocks are all covered by unchanged map tests is reported contained, naming those tests.
- D7: a test covering a block of an edited function that no map test covers is reported unique.
- D7: a judged test is never its own cover, and a covering map test whose function text changed since the map is not counted.
- D7: two judged tests covering only each other's blocks are never both reported contained.
- D7: a `//testtiming:keep` test, a `tier2` test, a test of an undeclared tier, tests past `redundancy_max_tests` and a timed-out test are listed as not judged with their reasons.
- D7: a package without a map is reported as not judged; one whose file hashes differ is judged and the report names the map commit, its age and the changed files.
- D7: a build failure, failing test or timed-out test in the step leaves the gate's result as its tiers decided it; a red tier gives a report that judges nothing.
- D7: an unknown report name in a gate key is refused with the known names.
- D8: the marker holds the findings when HEAD's judged packages and changed tests match the report, including after a commit touching only `_lym/`, and the no-report line otherwise.
- Review: D8's rubric text tells the reviewer to check assertions before filing a fold, delete or keep finding.
- Review: D9's background text states the package-scoped verdict, the map, the verb, the step and their bounds.

## PATTERN changes

- `PATTERN-test-speed` — changes: Writing or running a test: via `lym gate test` by gate key; one `tiergate.Require` tier, lowest that fits; it pins behavior no other test covers, which the gate's redundancy report checks; skips, clocks, `t.Parallel`, `//lym:seam` and `//lym:watch` per the background.

## Notes for the plan writer

- The study's scripts in `docs/research/speed/fabric-test-redundancy-2026-11-08/` show the runner shape the engine reproduces: one binary per package, each test run with `-test.run` anchored on its name and its own coverprofile.
- The gate key `lints` list in `internal/gateslot/template.yaml` is the model for the `reports` list.
- The green record in `internal/gatecli/green.go` shows how the gate already records which tree a result belongs to.
