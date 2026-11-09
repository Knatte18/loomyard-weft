# Add an advisory redundancy report step to lym gate test

A gate key in `gate.yaml` may now list `reports: [redundancy]`.
After the key's tiers and lints, `lym gate test` then judges the diff's new and changed Go tests against the stored coverage map.
It writes a report into the worktree's gate directory, so redundant tests stop piling up unnoticed.
The step is advisory: every error it meets is a line in the report, and the gate's exit code stays as its tiers decided it.

## What changes

- `internal/gateslot`: gate keys take a `reports` list.
  An unknown report name is refused, naming the known names and the way forward, as an unknown lint is.
  New keys `redundancy_max_tests` (default 40) and `redundancy_budget_min` (default 10) bound the step.
  `redundancy_build_timeout_min` and `redundancy_test_timeout_sec` now also bound the step's builds and tests.
  The shipped template lists the step on `webster`, `parent-align`, `darn-write` and `code-burler-fix`.
  A hub whose base `gate.yaml` tunes those keys gets the template's `reports` filled in by config load's `FillMissing`;
  `reports: []` on a key turns the step off.
- `internal/gatecli`: the step itself (`redundancy.go`, `redundancytests.go`, `redundancyreport.go`), called from `runTest` after the lints.
- `internal/testcover`: block keys, a static spawn scan that marks tests possibly running module code in another process, and verdict support for the greedy pass.
- `docs/overview.md` and the `gatecli`, `gateslot` and `testcover` package docs describe the step.

## How the step judges

1. It selects tests through the merge base with the parent branch.
   For each package holding a changed `_test.go` file, it reads the package's test files at the merge base through `fabricengine.CodeDirAt` and `fabricengine.CodeFileAt`.
   A current top-level test is taken when its text differs from the merge base or no such test exists there,
   so a renamed or moved file leaves its unchanged tests untaken and a deleted file adds none.
   A worktree with no parent branch judges nothing, and the report says so.
2. It drops, with the reason listed, `tier2` and `tier3` tests, tests of an undeclared tier, tests marked `//testtiming:keep` (with their keep reason) and tests the spawn scan flags.
   It keeps the first `redundancy_max_tests` tests in file then name order.
3. It takes one gate slot after the key's tiers release theirs.
   It builds a package-scoped coverage binary per package and runs each kept test alone and serially, building `lym` once if a `tier1` test runs.
   `redundancy_budget_min` counts from the moment the step holds its slot; once it is spent no further build or test starts, and the rest are listed as not judged.
4. Covers for a judged test are the map tests that still exist with the same function hash, minus the judged test itself, plus the other judged tests.
   The greedy pass runs slowest first, so two judged tests that cover only each other are never both reported contained.
5. The report gives each test as contained (with its covering tests, largest overlap first), unique, or not judged with a reason.
   Each judged package's line names its map's commit and age, and the files whose merge-base text differs from the map's hash.
   It records the function hash of every selected test and the hash of every Go file of each judged package.

A repeat call on a tree the existing report matches runs no build and no test.
A report with an error line, a package with no map or a failed-build map, or a test past the budget never skips the step, so a later call retries it.
A red tier gives a report saying so, and a hub whose `scoper` is not `go` gets a report that judges nothing.
A report that cannot be written is logged at Warn.

## Design decisions to know

- Per-test coverage needs one process per test, so the step cannot reuse a normal tier run.
- The step can let a redundant test through: a changed block counts as uncovered, a test changed since the map is not a cover, and tests past the cap or budget go unjudged.
- It can flag a test falsely: a map test counts as a cover while its own function text is unchanged.
  A change to a helper, a non-test callee, a package-level table, `TestMain` or `testdata` can stop that test reaching a block it once reached.
  Only a reviewer reading each finding against its covering tests bounds that.
- A new test marked `//testtiming:keep` is listed as not judged with its keep reason, so the reason is visible in the diff.
- No test launches a test binary or `go test`; the runner's tests use a stand-in `go` and fake profiles.

## To check by hand

- `internal/gatecli/redundancy.go`: a test killed by a cancelled call is listed as cancelled so the report is retried.
  Check that no cancelled step is kept as a complete report on a path the tests do not reach.
- The step has not been run against a real hub with built maps; run `lym gate test --gate webster` on a branch that adds a test and read the report.
