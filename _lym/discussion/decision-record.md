## Goal

An advisory redundancy report step in lym gate test, judging a diff's new tests against the map

Takes: testcover-engine's engine and map record; testmap-verb's CodeDirAt, CodeFileAt, timeouts and run-alone
Delivers: the `reports` list on gate keys with its unknown-name refusal and template entries on webster, parent-align, darn-write and code-burler-fix; the step at the lints' site with merge-base test selection, drops, cap, slot, budget, map covers, greedy pass and the report with its file and test hashes; keys redundancy_max_tests and redundancy_budget_min
Exports: the report reader and the tree match D8 runs on HEAD: D7 step 1 over a revision and the judged packages' file hashes

## Scope

The part changes these packages:

- internal/gatecli (about 800 changed lines)
- internal/gateslot (about 150 changed lines)
- internal/configengine (about 30 changed lines)
- internal/testcover (about 120 changed lines)

## Decisions

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
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - cancelled-step-kept-as-complete (class design, severity MEDIUM, open in rounds 1, 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

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
