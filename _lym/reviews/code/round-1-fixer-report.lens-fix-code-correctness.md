```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the fix-code-correctness lens over the F1 and F2 changes.

No findings.

**F1, `TestSyncOptionsNotifier` in `remoteteardown_test.go`:**
- **Fallback row:** `Notifier` is nil, so `SyncOptions.notifier()` returns `installedNotifier()`, which is the function `recordNotices` installed. Calling it records one line, so `installed()` returns 1 and `own` stays empty. That matches `wantInstalled=1, wantOwn=0`.
- **Shadow row:** `Notifier` is set, so `notifier()` returns it. `own` gets 1 and `installed()` returns 0, matching `0, 1`.
- **Fails with the defect in place:**
  - If `notifier()` ignored the installed one, the fallback row hits the nil `t.Fatalf`.
  - If it always returned the installed one, the shadow row fails on both counts.
  - If a nil `Notifier` shadowed the installed one, it returns nil and the fallback row hits the same nil check.
- **Isolation:** the test is serial, so no parallel test reads the installed notifier while it is swapped. `recordNotices` restores it through `t.Cleanup` on the subtest.

**F2, `GCAutoThresholdForTest` in `export_test.go` and the new subtest in `TestStoreHousekeeping`:**
- **Result:** the accessor returns the `gcAutoThreshold` field. `NewTopology` sets it to `defaultGCAutoThreshold` (6700), so the assertion holds.
- **Fails with the defect in place:** a `NewTopology` that left the field at zero yields 0, which is not 6700.
- **Literal:** the test repeats 6700 on purpose, to pin git's default rather than mirror the constant.
- **Callers:** `NewTopology`'s signature is unchanged, `Config{}` compiles as `Topology.cfg` is a `Config`, and the gate passed on tiers 0 and 1.

Tool calls made: none. My context already held every changed file.
