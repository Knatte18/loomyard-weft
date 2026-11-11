```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal, challenging the reviewer's proposals that the fixer carried out as proposed (F1, F2, F4, F5, F6, F7, F8, F9, F10).

**Findings**

1. **LOW** — `internal/gatecli/coveragescope_test.go`, `TestCoverageSelections`.
   - Summary: the test claims to pin caching but cannot fail without the cache.
   - Evidence: `openSelector` runs once because of the `opened` flag. It returns the same `setupMiss` pointer, so `coverageSelections` builds `Selection{Miss: setupMiss}` with the same pointer even with no cache. `reflect.DeepEqual(second, first)` therefore holds with the cache removed. Only the "outside a hub is a miss" half is pinned.
   - Cause: F2's proposed second row is indistinguishable without a reachable selector. `fabricengine.Open(nil)` dereferences nil, so the unreadable-maps row was rightly left out.
   - The failure message says "want the cached", which overstates what the test checks.

2. **LOW** — `internal/gatecli/scopes_integration_test.go`, new `--gate with scope coverage` row (F4's proposal).
   - Summary: neither half of the row depends on the key being `coverage`.
   - Evidence: under the `none` scoper, `computeScope` returns `scope{form, whole:true}` before any coverage code runs. A `diff` key would give the same `./...` and the same `computed` record. Under `go`, the `falls back to whole` path runs identically for a `diff` key.
   - What it does pin: `withGateScope` maps `coverage` to the computed form, and no `coverage:` line appears on a fallback. A regression that narrowed a whole fallback would show as a ` coverage: ` line.
   - Caveat: the row relies on the template giving `darn-write` the `coverage` scope.

3. **LOW** — `internal/gatecli/coveragescope.go`, `importerPackage` (F8's proposal).
   - Summary: the new guard-scan branch is unreachable from `computeScope`.
   - Evidence: `DeriveTargets` already ran `FindGuardTests` over every package in `scope.go:259`, and `computeScope` refuses on a guard-scan error at `scope.go:261`. An importer reaching `importerPackage` has therefore already scanned clean.
   - The branch is reachable only through `narrowCoverage` called directly, as the new row does, or if a file changes between the two scans.
   - This is harmless and matches the proposal, but the label fix covers a race and not a real scope-report case.

**Checked, no finding**

- **F9 rows:** the `testGraph()` map iteration order is harmless. `newModuleGraph` only fills maps, and `packageDirs` sorts the directories and `packageArgsOf` returns nil for an empty set. The expected `Packages`, `Importers` and `BinaryBuilders` are deterministic.
- **F10 prefix normalisation:** it only rewrites `lines[1]` when the lengths match and the prefix holds. A wrong label still fails the `slices.Equal` check.
- **F1 and F6:** the `MigrateConfig` prose in `gateslot/doc.go` still says `diff` for migrated keys. That describes the legacy-list migration output, not the template default, so it does not contradict the table.
- **F7:** the added `summary.md` bullet matches `fabricengine/diff.go:118`.
- **F5:** the `Recorded` assertion compares names. The map JSON round trip cannot reorder them, and `Recorded` is nil on a miss.

**Further tool calls (1 of 6 used)**
- `grep` of `newModuleGraph`, `packageDirs` and `packageArgsOf` in `internal/impactset/graph.go` and `derive.go`: my context lacked their ordering behaviour, which the F9 rows' expected slices depend on.
