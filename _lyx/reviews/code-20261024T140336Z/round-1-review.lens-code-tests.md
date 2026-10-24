```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing per behaviour the test that pins it.

**Per-behaviour list**

1. **Walk stops before climbing into a ceiling and returns `dir` unchanged.**
   - Pinned by `TestRepositoryRootOf/CeilingStopsTheWalkBelowIt` (`internal/cliwire/cliwire_test.go:239-258`), first assertion, `plain` with a `.git` at the ceiling.
   - The row would fail if the `slices.Contains(ceilings, parent)` check were dropped.

2. **A repository below the ceiling is still found.**
   - Pinned by the second assertion of the same row, `repo/src` resolving to `repo`.
   - It overlaps `SubdirectoryLiftsToRoot` and `NearestRepositoryWinsOverTopmost`, but only in walk mechanics.
   - Here it is a negative-side guard, and it runs with a non-empty ceiling list.
   - I judge it not redundant.

3. **The start directory is checked even when it is a ceiling.**
   - Pinned by the third assertion, `ceiling` resolving to `ceiling`.
   - It would fail if the check were moved before the `Lstat`.

4. **A relative or empty entry in the variable is ignored.**
   - Pinned by the leading-separator and `relative` entries in the `gitCeilingDirectories(...)` input.
   - It is only pinned indirectly. If relative entries were wrongly kept, the walk would be unaffected, because a relative path never equals an absolute parent.
   - The row therefore would not fail.
   - The summary at `_lyx/landing/summary.md` claims the row covers "a relative entry in the variable is ignored".
   - No assertion backs that claim.
   - Neither does the unit tested, since `gitCeilingDirectories` returns a slice that no assertion inspects.

5. **`RepositoryRootOf` reads `GIT_CEILING_DIRECTORIES` from the environment** (`internal/cliwire/paths.go:130-132`).
   - Pinned by no test.
   - Every row drives the unexported walk with parsed ceilings.
   - If `os.Getenv` were replaced by an empty string, or the variable name misspelled, no test fails.
   - The decision record chose this split deliberately to avoid `t.Setenv`.
   - The production wiring (the variable name and the call into the walk) therefore has no coverage, which is a gap under test economy.
   - The same acceptance criterion's "tier-0 tests pass with an empty `.git` in the temp root's parent" is only exercised indirectly through tmuxkit's env.

6. **Symlinked ceiling spelling is normalized so it still matches.**
   - Pinned by none.
   - The row uses `standalonestate.Normalize(t.TempDir())` for the ceiling, so the symlink path in `gitCeilingDirectories` is never exercised with a differing spelling.
   - This is a minor gap, since `Normalize` is already covered elsewhere.

7. **`tmuxkit` leaves `GIT_CEILING_DIRECTORIES` equal to the run directory.**
   - Pinned by `TestSetEnv` (`internal/testkit/tmuxkit/tmuxkit_test.go:52,62`).
   - The assertion was added to the existing loops.
   - It duplicates nothing and weakens nothing.
   - `t.Setenv` restores the variable.
   - It covers `setEnv`→`setTempEnv`.
   - The Windows path through `Main` (`setTempEnv` directly) is not covered, but the shared function is.

**Findings**

- **LOW, `design`:** `internal/cliwire/paths.go:130-132`, `RepositoryRootOf`'s environment read is pinned by no test.
  - A wrong variable name or a dropped `os.Getenv` call passes every test.
  - Evidence: `internal/cliwire/cliwire_test.go:239-258` calls only `gitCeilingDirectories` and `repositoryRootBelowCeilings`.
  - The decision record accepts this tradeoff, so it is arguably by design, hence LOW.
  - A subprocess-free test would need `t.Setenv`, which the record rules out.

- **LOW, `consistency`:** `internal/cliwire/cliwire_test.go:239-258` claims in `_lyx/landing/summary.md` (Tests section) that a relative entry is ignored.
  - The row feeds `"relative"` but asserts nothing that would fail if it were kept.
  - Either assert on `gitCeilingDirectories`'s result, or drop the claim.
  - This is also the NIT-level case of an unpinned behavior (item 4).

- No test-economy duplication found: the new row and the `TestSetEnv` change each cover behavior no existing test covers.

**Tool calls**
1. Read `internal/cliwire/cliwire_test.go` lines 184-240: the manifest only showed the diff hunk, not the existing `TestRepositoryRootOf` rows.
