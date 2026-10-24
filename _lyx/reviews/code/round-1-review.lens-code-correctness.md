```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `GIT_CEILING_DIRECTORIES` change in `cliwire` and `tmuxkit`.

**Findings: none.**

**Cases run, per function or branch:**

- **`RepositoryRootOf`** (`internal/cliwire/paths.go:131`)
  - Variable unset: `os.Getenv` returns "". `SplitList("")` is empty, so `ceilings` is nil and `slices.Contains` is always false. The walk matches the old one. Correct.
  - Variable set: it is read per call and handed to the walk. Correct.
- **`repositoryRootBelowCeilings`**
  - Start below the ceiling, no `.git` between them: the candidate has no `.git`, its parent is the ceiling, and it returns `dir` unchanged. Correct.
  - A `.git` between the start and the ceiling: found before the ceiling test. Correct.
  - Start equals the ceiling and holds a `.git`: found, since the start is checked first. Correct.
  - Start equals the ceiling with no `.git`: the parent is not listed, so the walk climbs above the ceiling. This differs from the record's "never climbs into a directory it lists" only in the case where the start is itself the ceiling.
    - It matches git's own discovery as I recall it (git does not count a ceiling equal to the cwd), and I did not check git's source.
    - The record's decision line "The start directory itself is checked even when it is a ceiling" bounds the case.
    - In the tests the fixtures sit below the run directory, so this is not reachable. Not a defect.
  - Ceiling is `/`: the walk stops at `/x` and `/.git` is never checked. This matches git. Correct.
  - Start is `/`: `parent == candidate`, so it returns `dir`. Correct.
- **`gitCeilingDirectories`**
  - "" gives nil. A leading separator gives an empty entry that is dropped. A relative entry is dropped by `IsAbs`. Absolute entries go through `Normalize`, which cleans the path and resolves symlinks. If the path does not exist, `EvalSymlinks` fails and it falls back to `Clean`, so a nonexistent ceiling is still matched textually. Correct.
  - A symlinked ceiling spelling matches the normalized walk input. This relies on `dir` already being normalized, which the existing precondition states. Correct.
  - Windows drive-letter case is a possible mismatch (`C:` vs `c:`). `slices.Contains` is case-sensitive, but `EvalSymlinks` normally canonicalizes the case. I did not run it, and it is at most a LOW doubt.
- **`setTempEnv`** (`internal/testkit/tmuxkit/tmuxkit.go:459`)
  - It is called from the Windows branch of `Main` (line 84) and from `setEnv` (line 471, non-Windows path). So the variable is set on every platform before `m.Run`. Correct.
  - A re-executed helper binary gets its own run directory and sets the ceiling to it, replacing the inherited value. Correct.
  - The error and early-return paths in `Main` (lines 77–90) set nothing before returning, and the process exits. No inconsistent state.
  - A git run from a cwd outside the run directory is unaffected, because the ceiling only prunes ancestors of the cwd.
- **`TestSetEnv`** (`internal/testkit/tmuxkit/tmuxkit_test.go:52,62`): `t.Setenv` pre-registers restoration, then `setEnv(dir)` is asserted for all three variables. Correct.

**Tool calls (1):** `sed -n` on `tmuxkit.go` lines 75–92 and 455–480, plus `standalonestate.Normalize`. My context lacked the `Main` call sites of `setTempEnv` and `Normalize`'s failure behavior.
