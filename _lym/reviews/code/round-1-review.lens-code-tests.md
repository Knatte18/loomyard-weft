```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour: the test that pins it**

- `impactset.HoldsTierTest` is pinned by `TestHoldsTierTest` in `internal/impactset/tiertests_test.go`. Its rows cover:
  - a listed tier, another tier, and no `Require`;
  - a benchmark, a fuzz target and a second file;
  - `TestMain` alone, a `Testable` helper, a method named like a test, and a non-test file;
  - a parse failure and an unreadable directory.
- The `tierTestTargets` narrowing is pinned by `TestTierTestTargets` in `internal/gatecli/testtargets_test.go`. Its rows cover:
  - a package without a test of the tier is dropped;
  - a covered tier keeps its package;
  - an empty result;
  - an import path is kept;
  - a list past `greenCommandBudget` returns the targets unnarrowed.
- `{test_targets}` rendering and quoting is pinned by the new row in `TestRenderTierCommand` in `internal/gatecli/tiers_unix_test.go`.
- `HoldsTargets` in `scopeInvocations` is pinned by the new `{test_targets}` row in `TestPlanTierRuns` in `internal/gatecli/tiers_test.go`.
- The `decodeTier` refusal of a `{test_targets}` command without `all_targets`, and the reworded message, are pinned by the new row in `TestLoadConfig` in `internal/gateslot/config_test.go`.
- The `expandPackages` refactor into a free function is pinned only by the existing green tests. I did not check that they run it.

**Findings**

1. **MEDIUM, design (test gap):** no test pins the wiring in `runInvocations`. The location is `internal/gatecli/tierrun.go`, the new block around lines 274–294 of the diff. These are unpinned:
   - the `{test_targets}` trigger;
   - `{targets}` staying whole while `{test_targets}` is narrowed;
   - the empty-case skip, with its `continue` and its stderr line;
   - the exit code `0` returned when every invocation is skipped.
   
   If the skip became a `break`, or ran the command with no arguments, `go test` would run the whole scope and no test would fail. `TestTierTestTargets` calls the helper directly, and the render test passes `testTargets` in by hand.

2. **LOW, design (test gap):** `tierTestTargets` has no test for its error fallback, where `expandPackages` fails and the function returns `targets` unnarrowed and logs a warning. The location is `internal/gatecli/tierrun.go`, the `err != nil` branch (diff lines 213–217). The test never makes `go list` fail.

3. **LOW, design (test gap):** a `...` pattern target is never tested through `tierTestTargets`. The test uses only `./a`, `./b` and an import path, so `expandPackages` never reaches `go list`. A broken `go list` expansion or build-tag handling in this path is uncaught. The location is `internal/gatecli/testtargets_test.go`, around line 141. Real tier commands use `./...`.

4. **LOW, design (test gap):** the `HoldsTargets` swaps in `internal/gatecli/green.go` are unpinned for a command that holds only `{test_targets}`. The swaps are in `subtract`, `carriesOverInert` and `recordPass`. The guard-step check in `planTierRuns`, `internal/gatecli/tiers.go`, line 241 of the diff, is unpinned too. The green and guard tests use only `{targets}` commands. Reverting any of these to `strings.Contains(..., "{targets}")` fails no test. That would give a `{test_targets}` tier `Whole` records and wrong subtraction.

**Test economy:** I found no duplicate tests, no per-helper test that the public surface already covers, and no assertion on static prose.

**Tool calls:** none. The manifest context held everything my lens needed.
