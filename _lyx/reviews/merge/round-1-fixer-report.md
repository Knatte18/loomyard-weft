# Fixer-report — merge, round 1

## Changes

- F1 (LOW), fixed with a changed location, commit `dff2d3f87`.
  The review named `internal/configcli/reconcile_integration_test.go:52`, but that test never runs `reconcile --layer default --apply`.
  The assertions went into the subtest that does, `reconcile --layer default migrates renamed keys in _board and commits them` in `internal/configcli/configcli_integration_test.go`.
  Before the successful apply it seeds an untracked `_lyx/orch/rules.md` in the board repo.
  After the apply it asserts that the existing HEAD file-list check still holds (only the loom config path), that `git status --porcelain --untracked-files=all` shows only `?? _lyx/orch/rules.md`, and that the rules file content is unchanged.
  No new test was added.
  `lyx gate test --tags integration ./internal/configcli` is green.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
