# Round 2 fixer-report

## Changes

- F1 (LOW), fixed as proposed: `backUpBaseline` in `internal/driftengine/capture.go` now renames the backup back to `baselineDir` when the metadata rename fails with an error other than `os.ErrNotExist`.
  It returns the metadata error, joined with the rollback's own error through `errors.Join` when the rollback fails.
  The doc comment states the rollback and the case where the move back fails too.
  No test row was added, as the review says.
  Commits: `2c7e22d7` (the fix) and a follow-up doc-comment commit from the self-check.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: the doc comment's "never left half-moved" overstated the guarantee when the rollback fails. Fixed as proposed: the comment now says the pair stays split when the move back fails too.

## Disputed

None.

## Deferred

None.
