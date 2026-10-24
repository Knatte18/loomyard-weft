# Honor GIT_CEILING_DIRECTORIES so a stray /tmp/.git can't break tests

Standalone wiring tests in `internal/webstercli` failed when an empty `/tmp/.git` existed.
`cliwire.RepositoryRootOf` walked up from a `t.TempDir()` fixture, found that `.git`, and took `/tmp` as the repository root.
The tests then refused the derived state directory as lying inside the standalone target.

Two changes close this off:

- `RepositoryRootOf` now honors `GIT_CEILING_DIRECTORIES` the way git's own repository discovery does.
  The walk never climbs into a listed directory, and a walk stopped there returns `dir` unchanged, as for a walk that found no `.git`.
  `dir` itself is still checked even when it is listed.
  Only absolute entries count, each normalized like the walk's input, split on the platform's path-list separator.
  With the variable unset, behavior is unchanged.
- `tmuxkit.Main` sets `GIT_CEILING_DIRECTORIES` to the process run directory, beside the `TMPDIR` redirect it already makes, on every platform.
  Every test package's `TestMain` runs through it, including packages that spawn no git and so skip `gitkit.HermeticGitEnv`.
  The run directory is the base `t.TempDir()` builds under, so neither the walk nor any git a test spawns climbs above it.

`PATTERN-test-isolation` records the new variable.

Closes #862

## Design notes

- `RepositoryRootOf` reads the variable and passes the parsed ceilings to an unexported `repositoryRootBelowCeilings`, which the tier-0 test drives directly.
  The existing `TestRepositoryRootOf` rows run in parallel, and `t.Setenv` would force the whole test serial.
- The variable can only shorten the walk, never redirect it, so the lift of an already-resolved absolute path stays free of cwd queries.
- A re-executed helper binary gets its own run directory and sets the ceiling to that.

## Not covered

- Whatever created `/tmp/.git` is not found, and it may keep writing outside its own temp directory.
  The ceiling only stops the tests from being affected by it.
- A test that deliberately places a repository at the run directory itself and walks from below it would now miss it; none is known.
- The new rows are tier 0; the broader `internal/webstercli` and `internal/burlercli` suites were not rerun by hand with an empty `.git` in the temp root's parent.
