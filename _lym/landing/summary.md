# Add an advisory redundancy report step to `lym gate test`

A gate key in `gate.yaml` may now list `reports: [redundancy]`.
A `--gate` call with that key runs the redundancy report step after its lints, whatever the key's scope.
The step judges the diff's new and changed Go tests against the per-package coverage maps that `lym gate testmap` builds.
It writes its verdicts to `redundancy.json` in the worktree's gate directory, so review can see a new test that adds no coverage.
The step is advisory: everything it runs into becomes a line of the report, and the exit code stays what the tiers and lints decided.

## What changes

- **Gate config (`internal/gateslot`).**
  `GateKey` gains `Reports`.
  The load refuses an unknown report name and names `redundancy` and `lym config gate` as the way forward, as it does for an unknown lint.
  The template lists the step on `webster`, `parent-align`, `darn-write` and `code-burler-fix`.
  Config load already fills a template key into a tuned `gates` entry that lacks it.
  So a hub base `gate.yaml` that tunes one of those keys without `reports`, as this hub's does, still gets the step, and `reports: []` turns it off.
  Two new cross-pair keys, each with a default and loaded through `configengine`: `redundancy_max_tests` (40) and `redundancy_budget_min` (10).
  The two existing redundancy timeouts now bound the step as well.
  No existing key changes meaning.
- **The step (`internal/gatecli`).**
  - *Selection.*
    The step diffs the working tree from the merge base with the parent branch (`parentMergeBase`, then `impactset.ChangedPaths`).
    For each package that holds a changed `_test.go` file, it reads the package's test files at the merge base through `Fabric.CodeDirAt` and `Fabric.CodeFileAt`.
    It takes each current top-level test whose name is missing at the merge base or whose text differs there.
    A test file moved within its package adds none of its unchanged tests, and a deleted test file adds none.
  - *Drops and cap.*
    The step drops a tier2-or-higher test, a test of a tier the hub doesn't declare, a `//testtiming:keep` test (listed with its reason) and a test the spawn scan marks as out of process.
    Each drop is listed with its reason.
    It then keeps the first `redundancy_max_tests` tests in file-then-name order and lists the rest.
  - *Maps.*
    A package with no map, an unreadable map or a map that records a build failure is not judged.
  - *Runs.*
    The step takes one gate slot after the tiers have released theirs.
    `redundancy_budget_min` starts counting once the slot is held, and the wait for the slot is unbounded.
    Per package, the step builds the coverage binary under `HolderEnv` and runs each test alone under its tier's `env`.
    It builds `lym` once if a tier sets `prebuilt_lym`.
    Once the budget is spent, the step starts no further build or test and lists every test it didn't run.
  - *Verdicts.*
    `testcover.JudgeAgainst` runs the greedy pass with every still-valid map test kept.
    A map test is still valid if a test with its name and function hash still exists and isn't itself being judged.
    A test the pass drops is *contained*, listed with its covering tests; every other judged test is *unique*.
  - *Report.*
    The report records the function hash of every selected test and the Go file hashes of each judged package.
    Each judged package's line names the map's commit and age, plus the files whose text at the merge base no longer matches the map.
  - *Repeat calls.*
    A repeat call on a tree that matches the report runs no build and no test.
    The step always runs again when the report holds an error line, a package without a usable map, or a test the budget left unrun.
  - *Reports that judge nothing.*
    A call where any tier failed, a call outside a hub, a hub whose `scoper` isn't `go`, and a worktree with no parent branch each get a report that judges nothing and says why.
  - *Exports for the review task.*
    `ReadRedundancyReport` reads a worktree's report.
    `RedundancyReportMatches` checks a report against a commit by redoing the selection over the commit's tree and comparing the judged packages' file hashes there.
- **Engine (`internal/testcover`).**
  `ParseTestFuncs` parses the tests of one file's source, and `TestFunc` gains `File`.
  `BlockKey.String` and `CoveredKeys` key a run's covered blocks the same way a map keys them.
  `JudgeAgainst` is `Judge` with an extra set of kept tests that cover blocks but are never judged or removed.
  `Judge` now delegates to it.
- `docs/overview.md` and the package docs of `gatecli`, `gateslot` and `testcover` describe the step and the new keys.

## Design decisions

- `internal/configengine` needed no change.
  The existing `yamlengine.FillMissing` call already fills a template key into a tuned per-name `gates` entry.
  A `gateslot` load test now pins that for `reports`, including `reports: []`.
- "When a tier is red" means any run of the call failed before the lints (tier, vet, guard or build).
  A lint finding alone doesn't stop the step from judging.
- A test with no `tiergate.Require` is judged as tier0, the tier it runs in.
- A failing or timed-out test is listed as not judged but doesn't force a re-run on an unchanged tree.
  Only error lines, unusable maps and budget-unrun tests do, as the decision record lists.
- A map test is excluded as a cover whenever a judged test in the same package has its name.
  This covers "minus the judged test itself" even when the map already holds the new test's text.
- Outside a hub, a key that lists the step gets a report saying no coverage map applies, since the maps live in the hub.
- The report sits beside the green record as `redundancy.json` in the worktree's gate directory.
  It is read and written through `internal/state`, keeping keys it doesn't declare (`PATTERN-tolerant-records`).

## Tests

- The new tests never launch a test binary or `go test`.
  The step's integration test (`TestRedundancyStep`, tier1) uses the `lym gate testmap` stand-in for `go`, which also stands in for the coverage binaries.
- The integration test runs over a hubforge pair diffing from the prime and covers:
  - contained and unique verdicts, and a stale map test not counted as a cover;
  - two judged tests that cover only each other;
  - every drop and not-judged reason;
  - the moved and deleted test files;
  - map drift, commit and age;
  - re-running while a package has no usable map;
  - skipping on a matching tree (no coverage build starts);
  - `RedundancyReportMatches` across commits;
  - a spent budget starting no build;
  - the three reports that judge nothing.
- `gateslot` load rows cover the unknown report name and its refusal, the new keys' bounds, and the template's `reports` filled into a tuned `webster` with `reports: []` honoured.
- `testcover` rows cover `JudgeAgainst`, `CoveredKeys` and the new `TestFunc.File`.
- No test pins that a failed report write is logged at Warn and leaves the result unchanged.
  Capturing the logger is process-wide and would make the step's test serial.

## To check by hand

- Run `lym gate test --wait --gate darn-write` in a task worktree whose packages have maps from `lym gate testmap`, and read `.lym/gate/redundancy.json`.
  This confirms that a real coverage build and real profiles key the same blocks as the map, which the stand-in tests can't show.
