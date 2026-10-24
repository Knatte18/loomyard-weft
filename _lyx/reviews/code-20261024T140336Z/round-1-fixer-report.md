# Fixer-report — code round 1

## Changes

- F1 (fixed as proposed): `internal/cliwire/cliwire_test.go` gains the serial `TestRepositoryRootOf_ReadsGitCeilingDirectoriesFromTheEnvironment`, which sets `GIT_CEILING_DIRECTORIES` through `t.Setenv` and asserts `RepositoryRootOf` on a plain directory below a ceiling holding a `.git` returns it unchanged. The parallel rows are untouched.
- F2 (fixed as proposed): the `CeilingStopsTheWalkBelowIt` row asserts `gitCeilingDirectories` returns exactly `[]string{ceiling}` through `slices.Equal`.
- F3 (fixed as proposed): `pattern/PATTERN-test-isolation.md` states that `tmuxkit.Main` also sets `GIT_CEILING_DIRECTORIES` to the run directory on every platform.

F1 and F2 touch one file and share a commit (`327f0d523`); F3 is `b700606ec`.
`lyx gate test ./internal/cliwire --tier 0` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings; one stale file-header remark (the header names only the ResolveStandalone tests as serial) was noted by the fork and not raised as a finding.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
