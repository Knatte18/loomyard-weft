# testcover: selector takes a target, compares own files, checks stamp

The coverage selector in `internal/testcover` now serves the gate and a later upkeep plan with one set of rules.
Before, it read the working tree only and missed on any change to the package's own Go files, so a package's own edits always ran the package whole.

## What changes

- `NewSelector` takes a `Target`: the working tree, or a commit read through `fabricengine.Fabric.CodeFileAt`.
  The working-tree file list moved from `SelectorConfig.WorkingFiles` to `Target.Files`.
- A package's own non-test Go files are compared by function, as a linked package's are.
  A test is affected when its blocks cover a changed own function, or its recorded functions cover a changed linked one.
  `MissOwnFiles` is gone.
- A package's own test files are compared with every top-level `Test` function set apart (`compareTestSources`).
  A `Test` function whose text changed, or that is new, is affected whatever the map records for it.
  A removed one is neither affected nor a miss.
  Any other change, such as a helper, `TestMain`, an `init`, a package-level declaration or the package clause, is a miss (`MissTestFile`).
- A changed non-Go file under the package, `testdata` and its Go files included, is a miss unless `inert_paths` exempts it and no `go:embed` pattern names it.
- A map's `EnvironmentStamp` that differs from the current one is a miss (`MissEnvironment`).
  The detail names the differing fields.
  `StampModuleFiles` fills the `go.mod` and `go.sum` hashes from the target's own tree, so a commit target stamps its own files.
- A `MapTest` entry may record an `Outcome` (`failed`, `timed out`, `skipped`, `covered nothing`) or a spawn verdict instead of a passing measurement.
  The selector reads such an entry as no recorded test, and `Selection.Recorded` holds only passing measurements.
  Spawn-marked tests are no longer selected on any change; the gate's union runs them as tests the map has no measurement for.
- A build-failure record beside test entries no longer misses the map.
  A map whose test entries name an out-of-range block now misses as unreadable.

`internal/gatecli` builds the current stamp through `stampEnvironment`, which `lym gate testmap` and the coverage scope now share.
Both stamp the toolchain, the digest of the mapped tiers' command and env before `{parallel}`, and the digest of the hub's `env.yaml` variables.
`mappedTierConfigs` collects the tiers for both.
`openSelector` reads `go env` under the gate's environment and passes the working tree as target.
`coverageUnion` drops its separate marked-test set, since `Recorded` already excludes those tests.

## Design decisions

- Under the gate a package the diff touches still runs whole, so the own-file rules narrow only a change that reached the base after the map's commit.
- A changed `Test` function selects its own test alone; test files are not instrumented, so nothing finer can be matched.
- A test file's import change is no miss when the test binary already links the import.
  An added import must be the package's own, in the map's linked set, or in the standard library.
  A removed import must be the package's own or in the standard library.
  Any other import change in a test file is a miss.
  This narrows the rule that every change outside a `Test` function misses; imports are the one exempted declaration.
  Without it, each new test file would force a whole rebuild of its package.
- A tier's parallel count stays out of the stamp, since it changes no test's coverage.

## To check by hand

- `internal/testcover/doc.go` and the miss list in the package doc: the claim about test-file imports (summary-test-file-import-claim-stale) should match the allowance in `packageChanges` in `internal/testcover/testselect.go`.
- The test-file import allowance (test-file-import-allowance-unrecorded) departs from the original rule that any non-`Test` change in a test file misses; confirm the departure is intended and recorded where the rule is stated.
- No regression check ran, and no integration suite ran for this task.
  The unit tests cover the selector, `compareTestSources`, `stampEnvironment` and `mappedTierConfigs`, plus a unix integration test of the coverage scope.
