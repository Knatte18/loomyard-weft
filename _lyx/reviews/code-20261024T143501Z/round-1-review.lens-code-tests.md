```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

1. **Tag scan, positive case** (`tagTiers`/`tagAddsFile`, `scope.go:384-430`): pinned by `scopes_integration_test.go:229`. The tier1 file in `lib` makes the call vet `./lib` and `.` under `tier1`.
2. **Tag scan, negative case (tag adds no file):** pinned only implicitly. The existing `--card` rows at `scopes_integration_test.go:205/234/265/357` have no tagged file in their fixture. They assert exact invocation lists with no vet run, so a scan that over-selected would fail them. No row isolates the case.
3. **Tag scan, tier0-tagged file:** a `//go:build tier0` file is selected for tier0, which the listed tier then skips. The failure row at `:243` exercises this scan path, but nothing asserts that tier0 is not vetted.
4. **Vet runs after the listed tiers and before the guard step, per-module `go vet -tags tierN` invocations:** `tiers_test.go` row "vets each tag tier no listed tier runs..." pins the plan order. Integration `wantArgs` at `:229` pins the real order: listed tier, two vet invocations, then the guard invocation.
5. **Listed tier or tier covered by a listed tier is not vetted:** `tiers_test.go` row "vets no tier a listed tier covers" pins both. `tagTiers` `[0,1]` with tier1 listed leaves neither vetted.
6. **`none` scoper plans no vet run:** `tiers_test.go` row "under the none scoper plans no vet run".
7. **Non-`--card` forms plan no vet run:** none. `tagTiers` is only populated on the card branch, so a missing `sc.form == scopeCard` check would change nothing observable. I don't count this as a gap.
8. **Failing vet:**
   - Pinned by integration `:241`: non-zero exit, a `tier1 vet ` summary line with a non-zero exit, `undefined: helper`, and `failed runs: vet tier1`. This covers the `runLabel` vet case and the scope form `vet`.
   - A passing vet is pinned by the exit-0 check on the summary line at `:229`.
9. **A failed vet stops no later run:** weakly pinned. The failure test asserts neither the guard run's presence nor its invocation list.
10. **`tierNumbers` excludes vet runs (`run.go:303`):** no test pins this. The only assertions on `TierList` (`scopes_integration_test.go:156`, `test_integration_test.go:620/939`) don't involve a `--card` vet. A vet tier number leaking into `tierList` would go unnoticed. In the failing-vet row, one added assertion on the run record's `TierList` equal to `[]int{0}` would cover it.
11. **Vet run takes part in the green record under its own fingerprint, carries when green (`green.go` `recordPass`, `reportCarried`):** none. Nothing runs `--card` twice on a clean tree and expects a carried vet. `green_integration_test.go` and `wantCarried` use `--all`. The decision record names this as a requirement (PATTERN-gate-carry). It is also the least obvious path: `recordPass` handles the vet tier with empty `Dir` and `AllTargets`. This is a coverage gap.
12. **`vetStepRun` refusals (undeclared tier, unquotable `goBinary`):** none. The error paths are untested, but the guard step's equivalent has the same shape and `planTierRuns` already refuses an undeclared tier earlier. LOW at most.
13. **Env, parallel and `timeout_min` inherited from the tier:** none. The run is built from `tier.Env`, `Parallel` and `TimeoutMin`, and no assertion reads them back.
14. **Helper-scoped assertions:** the `realGo` doc-comment change in `test_integration_test.go` needs no test.

**Test economy**

- No new test duplicates an existing one. The two integration subtests cover distinct behaviours (pass and fail). The three `tiers_test.go` rows are distinct.
- The new integration tests don't leak state. `edit` restores or removes each file in `t.Cleanup`, so the later timeout row at `:263` and the none-scoper row at `:357` still see an untagged `lib`.
- The success test (`:229`) and the failure test (`:241`) both rely on the fixture's helper names. This is not a defect.

**Findings**

1. **MEDIUM, design (coverage gap)**, `internal/gatecli/green.go:344-370` and `run.go:372-389`.
   - Summary: a vet run's green-record write and carry (own fingerprint, `reportCarried` with `ScopeForm` `vet`) is a behaviour the diff adds that no test covers.
   - Evidence: no `--card` test runs twice on a clean tree, and `wantCarried` (`scopes_integration_test.go:369-399`) only uses `--all`.
   - Suggested coverage: extend the vet-pass row (`:229`) or `green_integration_test.go` with a second clean-tree call asserting the vet is carried.
2. **LOW, design (coverage gap)**, `internal/gatecli/run.go:303`.
   - Summary: `tierNumbers` now excludes vet runs, and no assertion would fail if it did not.
   - Evidence: the run-record `TierList` assertions at `scopes_integration_test.go:156` and `test_integration_test.go:620/939` don't involve a `--card` vet.
   - Suggested coverage: add a `TierList` assertion to the failing-vet row.
3. **LOW, design (coverage gap)**, `internal/gatecli/scopes_integration_test.go:241-261`.
   - Summary: nothing asserts that a failing vet leaves the guard step running, the "stops no later run" criterion. The failure row checks only the exit, summary line, error text and closing line.
   - Suggested coverage: assert `guardInvocation` is the last tier invocation in the same row.

**Tool calls (3)**
1. `grep` of the `gatecli` test files for `tagTiers`, `tierList`, `vetStepRun` and tags, and for carried assertions: to find existing coverage the manifest lacked.
2. `grep` of the `--card` rows and the `wantArgs` definition: to judge the negative-scan and no-vet-for-other-forms coverage.
3. `sed` of the `edit` helper and the neighbouring rows: to check the new rows' cleanup doesn't affect later rows.
