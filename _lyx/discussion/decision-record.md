# Decision record — webstercli standalone tests fail when an empty /tmp/.git exists

## Goal

Tests that build standalone fixtures under `t.TempDir()` resolve the repository root inside the fixture or not at all, never at a stray `.git` above the test binary's temp directory, such as an empty `/tmp/.git`.

## Scope

- `internal/cliwire`: `RepositoryRootOf` stops its upward walk at the directories `GIT_CEILING_DIRECTORIES` names, as git's own repository discovery does.
- `internal/testkit/tmuxkit`: `Main` sets `GIT_CEILING_DIRECTORIES` to the process run directory, beside the `TMPDIR` redirect it already makes, so every test binary under it stops both the walk and any git it spawns at that directory.
- Tests: a `TestRepositoryRootOf` row for the ceiling, and a `TestSetEnv` assertion for the variable.
- Out of scope: finding whatever created `/tmp/.git`; its origin is unknown and no code path in the repository is identified as writing it.

## Decisions

### Honor GIT_CEILING_DIRECTORIES in RepositoryRootOf

Decision: the walk never climbs into a directory `GIT_CEILING_DIRECTORIES` lists; when the next parent is a ceiling, the walk ends and returns `dir` unchanged, as for a walk that found no `.git`.
The start directory itself is checked even when it is a ceiling.
Only absolute entries count, each normalized the way the walk's input is, and the list splits on the platform's path-list separator.
Rationale: the walk mirrors git's repository discovery, so honoring git's ceiling keeps lyx's notion of the enclosing repository equal to git's.
Bound: the variable can only shorten the walk, never move it elsewhere; with it unset, behavior is unchanged.

### Set the ceiling in tmuxkit.Main

Decision: `tmuxkit.Main` sets `GIT_CEILING_DIRECTORIES` to the run directory in the same place it sets `TMPDIR`, on every platform.
Rationale: every test package's `TestMain` already runs through `tmuxkit.Main`, including packages that spawn no git and so skip `gitkit.HermeticGitEnv`, such as `internal/cliwire`; the run directory is exactly the base `t.TempDir()` builds under.
A re-executed helper binary gets its own run directory and sets the ceiling to it in turn.

### Keep the walk free of injected state at the public surface

Decision: `RepositoryRootOf` reads the variable and hands the parsed ceilings to an unexported walk, which the tier-0 test drives directly.
Rationale: the existing `TestRepositoryRootOf` rows run in parallel, and `t.Setenv` would force the whole test serial.

## Constraints

- `PATTERN-cwd-resolution`: the walk stays a lift of an already-resolved absolute path, never a cwd query.
- `PATTERN-test-speed`: the new row stays tier 0 and parallel.
- `PATTERN-testkit`: the tmuxkit change stays inside the kit's existing environment setup.
- `PATTERN-comment-line-breaks`: one sentence per line in every Go comment touched.

## Auto-mode assumptions

- No interview: the board entry names `GIT_CEILING_DIRECTORIES` as an acceptable fix, and this record takes it.
- Honoring the variable in production is wanted: without it, setting the variable in tests would not reach `RepositoryRootOf`.

## Open risks

- The process that created `/tmp/.git` is not found; another stray marker above `/tmp` would still be caught by the ceiling, but the writer keeps writing outside its own temp directory.
- A test that deliberately places a repository at the run directory itself and walks from below it would now miss it; none is known.

## Acceptance criteria

- With a `.git` above the ceiling and none between the start and the ceiling, the walk returns the start unchanged.
- A `.git` between the start and the ceiling is still found.
- `tmuxkit`'s environment setup leaves `GIT_CEILING_DIRECTORIES` equal to the run directory.
- `internal/webstercli`, `internal/burlercli` and `internal/cliwire` tier-0 tests pass with an empty `.git` in the temp root's parent.

## Notes for the plan writer

- `cliwire`'s doc on `RepositoryRootOf` names its precondition and its no-spawn rationale; extend it with the ceiling rule.
