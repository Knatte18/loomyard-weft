# Stop repository-root walks at GIT_CEILING_DIRECTORIES

Fixes #862.

An empty `/tmp/.git` made `internal/webstercli`'s standalone wiring tests fail with "derived state directory ... lies inside the standalone target /tmp".
`cliwire.RepositoryRootOf` lifts a standalone target to the nearest ancestor holding a `.git`, and a fixture built under `t.TempDir()` has no `.git` of its own, so the walk climbed out of the test's temp tree and stopped at `/tmp`.
`cliwire`'s own `NoRepositoryAboveIsReturnedUnchanged` row was exposed the same way.

## What changes

- `cliwire.RepositoryRootOf` honors `GIT_CEILING_DIRECTORIES` as git's repository discovery does.
  The walk never climbs into a listed directory, and a walk stopped there returns its input unchanged, as a walk that found no `.git` does.
  The start directory is checked even when it is listed.
  Only absolute entries count, normalized through `standalonestate.Normalize` to match the walk's already-normalized input.
  With the variable unset, behavior is unchanged.
- `tmuxkit.Main` sets `GIT_CEILING_DIRECTORIES` to the test binary's lyxtemp run directory, beside its existing `TMPDIR` redirect.
  Every test package already runs through `tmuxkit.Main`, so fixture walks and the git processes tests spawn both stop at the run directory, whatever sits above it.

## Design decisions

- The ceiling is git's own variable rather than a lyx-specific one, so lyx's notion of the enclosing repository stays equal to git's, and spawned git honors the same bound.
- The ceiling is set in `tmuxkit` rather than `gitkit.HermeticGitEnv`, because packages that spawn no git, such as `internal/cliwire`, skip the hermetic call but still walk the filesystem.
- The walk is split into an unexported `repositoryRootBelowCeilings` taking parsed ceilings, so the new tier-0 row drives it without `t.Setenv` and `TestRepositoryRootOf` stays parallel.
- The writer of `/tmp/.git` is not identified; no code path in the repository was found writing it.
  The ceiling makes the tests immune to any such marker above the run directory.

## Tests

- `TestRepositoryRootOf` gains `CeilingStopsTheWalkBelowIt`: a `.git` at the ceiling is not reached from below, a repository below the ceiling is still found, the start is checked when it is itself the ceiling, and a relative entry in the variable is ignored.
- `TestSetEnv` asserts that `tmuxkit`'s environment setup leaves `GIT_CEILING_DIRECTORIES` equal to the run directory.
