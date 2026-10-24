# Stop repo-root walks and test git at the test run directory

An empty `/tmp/.git` made the standalone wiring tests in `internal/webstercli` treat `/tmp` as the repository root, and they failed with "derived state directory ... lies inside the standalone target".
Test fixtures live under `t.TempDir()`, so `cliwire.RepositoryRootOf` walked up out of the fixture into the stray marker.

`RepositoryRootOf` now honors `GIT_CEILING_DIRECTORIES` as git's own discovery does.
`tmuxkit.Main` sets that variable to the process run directory, beside its `TMPDIR` redirect, so a walk or a spawned git from a temp fixture never climbs above the directory `t.TempDir()` builds under.

Closes #862

## Design decisions

- The walk never climbs into a listed directory.
  When the next parent is a ceiling, it ends and returns the start unchanged, as for a walk that found no `.git`.
  The start directory itself is still checked when it is listed, as in git.
- Only absolute entries count, split on the platform's path-list separator and normalized as the walk's own input is, so a symlinked spelling still matches.
  Relative and empty entries are dropped, as git drops them.
- The variable can only shorten the walk.
  With it unset, behavior is unchanged.
- `RepositoryRootOf` reads the variable and hands the parsed ceilings to the unexported `repositoryRootBelowCeilings`.
  The tier-0 test drives that function directly, because the existing `TestRepositoryRootOf` rows run in parallel and `t.Setenv` would force the test serial.
- The ceiling is set in `tmuxkit.Main` rather than per test package.
  Every package's `TestMain` already runs through it, including packages such as `internal/cliwire` that spawn no git and so skip `gitkit.HermeticGitEnv`.
  A re-executed helper binary gets its own run directory and sets the ceiling to it in turn.
- `pattern/PATTERN-test-isolation.md` records the new variable beside the `TMPDIR` redirect.

## Not addressed

Whatever created `/tmp/.git` is not identified, and no code path in the repository is known to write it.
The ceiling stops it from affecting the tests, but the writer may still write outside its own temp directory.

A test that deliberately places a repository at the run directory itself and walks from below it would now miss that repository.
None is known.

## Checks by hand

- Create an empty `/tmp/.git` and run the tier-0 tests of `internal/webstercli`, `internal/burlercli` and `internal/cliwire`; they should pass.
  This was not run against a real stray `.git`.
