```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, listing per behaviour the test that pins it, and applying test economy to the amendment diff.

**Per behaviour, the test that pins it**

- Walk stops below a ceiling and returns `dir` unchanged: `TestRepositoryRootOf/CeilingStopsTheWalkBelowIt` (`internal/cliwire/cliwire_test.go`, around line 241). The `plain` assertion would fail if the `slices.Contains(ceilings, parent)` check were dropped.
- A repository below the ceiling is still found: the same subtest, the `repo/src` assertion.
- The start directory is checked even when it is a ceiling: the same subtest, the `ceiling` start assertion.
- Relative and empty entries are dropped and absolute entries are kept: the same subtest, the `slices.Equal(ceilings, []string{ceiling})` assertion on a list with an empty and a relative entry. If `filepath.IsAbs` were removed, `relative` would appear in the result and the assertion would fail.
- Entries are normalized, so a symlinked spelling matches: none. The fixture passes `standalonestate.Normalize(t.TempDir())` as the ceiling, so it is already normalized and `Normalize` inside `gitCeilingDirectories` is never exercised. This is a gap in behaviour the diff adds. It is LOW at most, because the temp root is rarely a symlink on Linux, and macOS `/var` to `/private/var` is where it would matter.
- `RepositoryRootOf` reads `GIT_CEILING_DIRECTORIES` from the environment: `TestRepositoryRootOf_ReadsGitCeilingDirectoriesFromTheEnvironment`. It would fail if the `os.Getenv` read were dropped or misnamed.
- Unset variable leaves behaviour unchanged: the pre-existing `TestRepositoryRootOf` subtests, run with the variable inherited from `tmuxkit.Main` (the run directory). They cover the unset case only incidentally, since the variable is set to the run directory under `tmuxkit`. Acceptable.
- `tmuxkit.Main` sets `GIT_CEILING_DIRECTORIES` to the run directory: `TestSetEnv` (`internal/testkit/tmuxkit/tmuxkit_test.go`, lines 52 and 62). The variable is added to the restore list and to the assertion list.
- `setTempEnv` is reached via `setEnv`: `TestSetEnv` calls `setEnv`, which calls `setTempEnv` (`tmuxkit.go:469-471`).
- Path-list splitting on multiple entries: no row passes two absolute entries. `filepath.SplitList` is stdlib and the empty-entry row exercises the splitting. Not a finding.

**Test economy**

- `TestRepositoryRootOf_ReadsGitCeilingDirectoriesFromTheEnvironment` covers the env read, which no other test covers, so it is not a duplicate. It is serial with a stated reason. Held.
- The `CeilingStopsTheWalkBelowIt` subtest bundles four assertions into one subtest. They are different aspects of one behaviour, so the one-test-per-behaviour rule holds. Held.
- The `TestSetEnv` change extends the existing test rather than adding a new one. Held.

**Findings**

1. LOW, class `design`, `internal/cliwire/cliwire_test.go` (`CeilingStopsTheWalkBelowIt`, lines 241-262) and `internal/cliwire/paths.go:148-156`. The `standalonestate.Normalize` call in `gitCeilingDirectories` has no test that would fail if it were removed. The fixture's ceiling is already normalized, so the symlinked-spelling behaviour that the doc comment claims is unpinned.

No other findings.

**Tool calls:** none. The manifest and the inherited diff and test reads already held every file this lens needed.
