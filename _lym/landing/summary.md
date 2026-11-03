# Preflight: ignore lym's friction notes and archives under _lym

The worktree-clean check at Preflight refused a run when the dirty paths were lym's own friction state.
A halt wrote a note into `_lym/loom/friction/`, a reflection then deleted or moved that note into an archive, and the resulting uncommitted change blocked the next step.
So one halt's note caused the next halt, and only a manual `lym fabric commit` cleared it.

The check now skips the loom friction directory and its timestamped reflection-archive siblings, alongside the run-records root it already skipped.
Everything else under `_lym` still counts.
The settle logs under `_lym/shed/<slug>/` are covered by the run-records exemption that was already in place.

Closes #33

## Design decisions

- The exemption is the check ignoring lym's own state, not an auto-commit.
  Committing from inside Preflight would make a read-only precondition write history; the friction notes are bookkeeping in flight, never work a run could lose.
- `runStateExemptRels` in `internal/preflight/preflight.go` stays the one place that decides which lym state the check skips.
  It now returns the run-records root, the friction directory and a pathspec for the archives.
- The archives are named by a new `loomengine.LoomFrictionArchivesRelPattern`, the friction directory path plus `-*`.
  `fabricengine.Clean` already takes the weft exemptions as git pathspecs, and its doc comment now says globs are allowed.
- The pattern is a pathspec only: git's `*` matches across `/`, so it must not be used as a file path match.

## To check

- `TestCheckResolved_LymOwnStateIsExempt` in `internal/preflight/preflight_integration_test.go` covers a committed halt note deleted from the friction directory, an archive created beside it, and a new friction note, and still asserts a file elsewhere under `_lym` refuses.
  It is an integration-tier test; run it through `lym gate test ./internal/preflight` with the integration tier, since it was not run when this description was written.
