# Fixer-report, code round 1

## Changes

- F1: fixed as proposed in commit 1ae83afea.
  The `runCoreWithin` doc comment in `internal/gitexec/gitexec.go` now says every child gets the process environment less `GIT_DIR`, `GIT_WORK_TREE` and `GIT_OPTIONAL_LOCKS`, with `GIT_OPTIONAL_LOCKS=0` set.
- F2: fixed as proposed in commit 1ae83afea.
  The remote row of `TestRun_RemoteBounds` in `internal/gitexec/gitexec_test.go` also wants `GIT_OPTIONAL_LOCKS=0` in `gitErr.Stderr`.
  No new test function.

`lyx gate test ./internal/gitexec` is green at tier 0 and tier 1.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
