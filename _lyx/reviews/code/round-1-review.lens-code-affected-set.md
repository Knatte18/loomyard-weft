```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** Per change, the set I found and whether it is covered:

1. **`RepositoryRootOf` now reads `GIT_CEILING_DIRECTORIES`.**
   - Production callers: only `internal/cliwire/standalone.go:207`, whose input is already `Normalize`d. Covered.
   - Docs: the `RepositoryRootOf` doc in `paths.go` is extended with the ceiling rule. The `standalone.go:186` comment and `docs/benchmarks/test-redundancy.md:448` only name the function, so nothing in them goes stale. Covered.
   - Existing `TestRepositoryRootOf` rows and `NoRepositoryAboveIsReturnedUnchanged`: they run under `tmuxkit.Main`, so the ceiling is the run directory and `t.TempDir()` sits below it. They no longer climb out of the run directory, which is the intended fix. Covered.
   - Test packages that run under `tmuxkit.Main` but are not a `RepositoryRootOf` caller: unaffected.

2. **`tmuxkit.setTempEnv` now exports `GIT_CEILING_DIRECTORIES=<run dir>` for every test binary.**
   - `setTempEnv` is called from `Main` on Windows and from `setEnv` elsewhere. Both are covered, and `TestSetEnv` asserts the variable.
   - Spawned git, from every package whose `testmain_test.go` calls `gitkit.HermeticGitEnv()`: it inherits the ceiling. Fixtures sit in subdirectories of the run directory, and git checks the start directory even when it is listed, so discovery from a fixture is unchanged. Covered.
   - `internal/gitkit/hermetic.go` (`HermeticGitEnv`): from the lines I read, it builds the hermetic gitconfig. I did not see it touch ceiling variables, so I found no conflict. I did not read the rest of the function.
   - Tests that read `lyxtemp.Dir()` directly: `internal/gitexec/gitexec_test.go:458`, `internal/gitkit/rundir_integration_test.go:40` and `internal/gatecli/test_integration_test.go:1070`. I did not open them, so I could not rule out a `git init` at the run directory itself. See finding 1.
   - Test helper binaries re-executed from a test: each makes its own run directory and sets its own ceiling. The package doc covers this.
   - Doc edits: the `tmuxkit` package doc and the `setTempEnv` doc both carry the ceiling rule. Covered.

3. **The test edits.**
   - `TestSetEnv` restores `GIT_CEILING_DIRECTORIES` through `t.Setenv`, so it leaks no state.
   - The new `cliwire` row calls `repositoryRootBelowCeilings` directly and sets no environment. Covered.

**Findings**

1. **LOW, class `consistency`**, location `internal/gitexec/gitexec_test.go:458`, `internal/gitkit/rundir_integration_test.go:40` and `internal/gatecli/test_integration_test.go:1070`.
   - Summary: each of these three tests calls `lyxtemp.Dir()` itself, and the diff leaves them unchecked against the new ceiling.
   - Reason: a test that runs git, or the repository-root walk, with the run directory itself as the repository now misses its `.git` when started from below. This is a known open risk in the decision record.
   - Evidence: only the `lyxtemp.Dir()` call sites appear in the grep output, and I read no bodies. No evidence of breakage, so this is a coverage gap in my own reading.

**Tool calls**
1. Grep for `HermeticGitEnv` and `GIT_*` across `internal`, plus `RepositoryRootOf` outside tests: I needed the set of callers and the packages whose environment is touched.
2. `grep -n "func HermeticGitEnv"` on `internal/gitkit/*.go` (wrong path, no output), plus a read of `RepositoryRootOf`'s doc in `paths.go` and a grep for `TMPDIR` consumers: I needed the other readers of the environment variables that `setTempEnv` sets.
3. Grep for `lyxtemp.Dir()` callers in tests, plus the start of `HermeticGitEnv`: I needed tests that build on the run directory itself.
