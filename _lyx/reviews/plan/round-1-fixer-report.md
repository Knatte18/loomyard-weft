# Plan round 1 — fixer report

## Changes

- **F1** (MEDIUM, consistency), fixed as proposed in `_lyx/plan/10-restore-set-content-rows.md`, Intent.
  The premise holds: `git show 4ddcb5133` shows the removed `SetContentRows` doc comment ending "and the pane process retries at its next poll after any refusal" and the removed `doc.go` Bound sentence ending with the same retry clause, while `internal/statuspane` holds no `Report` or retry.
  - The `Engine.SetContentRows` bullet now restores the body unchanged and the doc comment unchanged but for its last clause, which is dropped, so the comment's last sentence ends after "so it can resize no strand".
  - The `doc.go` bullet now says the restored paragraph's Bound sentence ends after "so it can resize no strand", dropping the retry clause and its "report made before AddStrand" tail.

## Disputed

None.

## Deferred

None.
