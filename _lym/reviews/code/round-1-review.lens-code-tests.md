```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it.

**Per behaviour, the test that pins it**

1. `HolderEnv` puts `-count=1` first, then the base's GOFLAGS, then `-p`, then `-trimpath`.
   - Pinned by `TestLeaseEnv` holder rows in `internal/gateslot/pool_test.go:536-540`.
   - Those rows cover tier0Parallel above and equal to 0, an existing GOFLAGS value, and the lease and no-lease cases.
   - Covered.
2. `WithTestCacheOff` with no GOFLAGS gets the flag alone.
   - Pinned by the `cacheOffTests` row "no GOFLAGS" (`pool_test.go:559`).
   - Covered.
3. `WithTestCacheOff` takes the last GOFLAGS entry, writes a single entry, and puts it last.
   - Pinned by the row "the last GOFLAGS follows the cache-off flag" (`pool_test.go:560`).
   - Covered.
4. `WithTestCacheOff` with an empty `GOFLAGS=` gets the flag alone, with no trailing space.
   - Pinned by the row "an empty GOFLAGS" (`pool_test.go:561`).
   - Covered.
5. `WithTestCacheOff` leaves its input slice unchanged.
   - Pinned by the base-unchanged assertion in the same loop (`pool_test.go:568-570`).
   - Covered.
6. `WithTestCacheOff` matches the GOFLAGS key case-insensitively.
   - None. Every `cacheOffTests` row uses the exact `GOFLAGS` spelling.
   - Without `strings.EqualFold` a lowercase `goflags=` entry would be kept next to a second `GOFLAGS` entry, and no row would fail.
   - Finding below.
7. `runTier` applies `WithTestCacheOff` to every tier invocation, after the tier's own env is merged.
   - Pinned by the gatecli scenario tests in `internal/gatecli/test_integration_test.go`, which expect the stand-in to see `-count=1 -trimpath -p=3`.
   - The line 1093 case runs through an inherited slot.
   - The line 1907 case checks that the tier's GOFLAGS overrides the env module's.
   - The line 1932 case runs with no slot.
   - Covered.
8. The `-trimpath` note must still name the GOFLAGS without the added flag, because `trimpathLine` is called on `tierEnv` before `WithTestCacheOff`.
   - The scenarios at lines 1107 and 2180 build the expected line from `trimpathLine("tier0", []string{"GOFLAGS=-trimpath -p=3"})`, with no `-count=1`, and the check at `scopes_integration_test.go:217` does the same.
   - Moving the wrap ahead of `trimpathLine` would break them.
   - Covered.
9. `HolderEnv` reaches its non-tier callers: `cardverify`, `driftengine`, `cmd/testtiming` and `planglyph`.
   - `planglyph`'s `loadEnv` is pinned by `glyphchain_integration_test.go:660`.
   - `cardverify` and `driftengine` have no GOFLAGS assertion of their own; they inherit the `TestLeaseEnv` rows for `HolderEnv`. That is adequate, since a per-caller test would duplicate it.
   - Covered.
10. The doc and comment changes ("build cache" instead of "build and test cache", and the new "Test result cache" section) are static prose, so they are exempt.

**Findings**

- **LOW, design:** `internal/gateslot/pool_test.go:556-562` has no `cacheOffTests` row for a lowercase `goflags=` entry. The case-insensitive key match in `WithTestCacheOff` (`pool.go:449`) is therefore unpinned.
  - Fix: add a row such as `{"a lowercase goflags entry is replaced", []string{"goflags=-trimpath"}, []string{"GOFLAGS=-count=1 -trimpath"}}`.
  - It belongs in the existing table, not in a new test.

No test is redundant. The `cacheOffTests` rows and the gatecli scenarios cover different layers: the first cover the function's env handling, the second its wiring in `runTier`.

**Tool calls (1)**
- Grep for tests asserting GOFLAGS values outside `pool_test.go`. My context lacked the `cardverify`, `driftengine` and `testtiming` assertions and the trimpath-note tests.
