---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/cliwire/paths.go:130
    summary: RepositoryRootOf's read of GIT_CEILING_DIRECTORIES is pinned by no test, so a misspelled variable name or a dropped os.Getenv passes every test
    fix: In internal/cliwire/cliwire_test.go add one separate top-level non-parallel test, with a comment saying it is serial because of t.Setenv, that sets GIT_CEILING_DIRECTORIES to a seeded ceiling through t.Setenv and asserts RepositoryRootOf on a plain directory below a ceiling that holds a .git returns the directory unchanged. Leave the parallel TestRepositoryRootOf rows as they are.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/cliwire/cliwire_test.go:248
    summary: The change description claims the new row shows a relative entry is ignored, but no assertion fails if gitCeilingDirectories keeps the relative entry
    fix: In the CeilingStopsTheWalkBelowIt row, assert that the gitCeilingDirectories result equals []string{ceiling} (compare with slices.Equal), so keeping the relative or empty entry fails the row.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-pattern-fit
    location: pattern/PATTERN-test-isolation.md:38
    summary: The PATTERN-test-isolation background lists the variables tmuxkit.Main sets and omits the new GIT_CEILING_DIRECTORIES
    fix: In pattern/PATTERN-test-isolation.md, in the bullet at line 38 that says tmuxkit.Main points TMPDIR (and TMP and TEMP on Windows) at the run directory, add one sentence on its own line saying Main also sets GIT_CEILING_DIRECTORIES to that directory on every platform, so neither a repository-root walk nor a spawned git from a temp fixture climbs above it.
---

### [LOW] Environment read of GIT_CEILING_DIRECTORIES is untested

**Location:** `internal/cliwire/paths.go:130`
**Issue:** `RepositoryRootOf` reads the variable and hands the parsed ceilings to the walk. Every new row drives `gitCeilingDirectories` and `repositoryRootBelowCeilings` directly, so replacing the `os.Getenv` argument with an empty string, or misspelling the name, fails no test. The decision record chose the split to keep `TestRepositoryRootOf` parallel, and that choice is sound; what it leaves open is the one-line wiring, which a separate serial test can pin without touching the parallel rows.

### [LOW] Relative-entry claim has no assertion behind it

**Location:** `internal/cliwire/cliwire_test.go:248`
**Issue:** `_lyx/landing/summary.md` says the row covers "a relative entry in the variable is ignored". The row feeds `"relative"` and an empty entry but asserts only walk results. A relative path never equals an absolute parent, so keeping it would change no result and the row would still pass.

### [LOW] PATTERN-test-isolation background does not list the new variable

**Location:** `pattern/PATTERN-test-isolation.md:38`
**Issue:** The bullet describes what `tmuxkit.Main` sets in the test process environment. The diff adds `GIT_CEILING_DIRECTORIES`, which changes git discovery in every test package, and the background file does not mention it.

## Rejected

- code-affected-set finding 1 (the three tests that call `lyxtemp.Dir()`): rejected. The handler read the three sites (`internal/gitexec/gitexec_test.go:458`, `internal/gitkit/rundir_integration_test.go:40`, `internal/gatecli/test_integration_test.go:1070`) and none of them runs git or a repository-root walk from below the run directory. The lens itself found no evidence of breakage.
- code-tests item 6 (symlinked ceiling spelling not exercised): not raised. `Normalize` is covered where it is defined, and the row's use of it for the ceiling is the same call the production path makes.
- code-correctness note that `RepositoryRootOf` does not normalize `dir`: not raised. The existing doc states the precondition and the only production caller (`internal/cliwire/standalone.go:207`) normalizes first.
- Git's rule that an empty entry stops symlink resolution for later entries is not mirrored: not raised. The decision record limits the parse to absolute entries normalized like the walk's input, and the test environment never writes such a list.
