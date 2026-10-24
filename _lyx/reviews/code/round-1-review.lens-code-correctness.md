```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function or branch against concrete inputs. I found no defects.

**List**

- `repositoryRootBelowCeilings`, `.git` at the start directory: it returns the start. Correct. The `Lstat` runs before the ceiling check, so the start is checked even when it is listed.
- `repositoryRootBelowCeilings`, start `/c/plain/src` with ceiling `/c` holding a `.git`:
  - `src` and `plain` have no `.git`.
  - The parent of `plain` is `/c`, which is a ceiling, so the walk returns `dir` unchanged.
  - Correct. The `.git` at the ceiling is never reached, which matches git's rule that it never climbs into a ceiling.
- `repositoryRootBelowCeilings`, start `/c/repo/src` with `/c/repo/.git`: it finds `/c/repo` before the ceiling is relevant. Correct.
- `repositoryRootBelowCeilings`, start equal to the ceiling:
  - It checks the start and returns it if the start has a `.git`.
  - Otherwise the parent of the start is not a ceiling, so the walk continues upward. This matches the record, and my recollection of git's behavior.
- `repositoryRootBelowCeilings`, filesystem root:
  - Ceiling `/` with start `/x`: the parent `/` is a ceiling, so the walk stops. Correct.
  - Start `/`: `parent == candidate` ends the walk. Correct.
- `repositoryRootBelowCeilings`, empty or nil ceilings: `slices.Contains` is false, so behavior is unchanged from before. Correct.
- `gitCeilingDirectories("")`: `SplitList("")` returns an empty slice, so it returns nil. Correct.
- `gitCeilingDirectories`, value `":relative:/abs"`:
  - The empty and relative entries are dropped by `IsAbs`.
  - `/abs` is normalized.
  - Correct, and covered by the new row.
- `gitCeilingDirectories`, trailing separator, `..` segments or a symlinked spelling:
  - `Normalize` runs `EvalSymlinks` then `Clean`, which is the same form the production caller gives `dir`.
  - Correct.
- `gitCeilingDirectories`, a ceiling that does not exist: `EvalSymlinks` fails and `Normalize` falls back to `Clean`. It cannot match a real candidate, which is harmless.
- `RepositoryRootOf`, unset variable: it behaves as before. Correct.
- `tmuxkit.setTempEnv`:
  - Setting `GIT_CEILING_DIRECTORIES` to `dir` overwrites any operator value, which is acceptable for a test binary.
  - `dir` comes from `lyxtemp.Dir` and is the base `t.TempDir` builds under. Fixtures sit below it, and the ceiling only blocks climbing into `dir` from below it.
  - Git discovery from a cwd outside the run directory, such as the source tree, is unaffected.
  - A fixture repository at a subdirectory of `dir` is still found.
  - A re-executed helper sets its own run directory in turn, so correct.
- `setTempEnv` called on the Windows path in `Main`, before `m.Run`: it sets the same variable, and `SplitList` uses `;` there, so a single entry parses. Correct.
- `TestSetEnv` and the `CeilingStopsTheWalkBelowIt` row: both assert the intended values. I ran no case that gives a wrong result.

**Notes, not findings**

- The public `RepositoryRootOf` does not normalize `dir` itself, while the ceilings are normalized.
- A caller passing an un-normalized `dir` could therefore miss a ceiling.
- The only production caller (`internal/cliwire/standalone.go:207`) normalizes first, and the existing doc states a precondition on `dir`, so I raise no finding.

**Tool calls:** none. The inherited context already held every symbol I needed.
