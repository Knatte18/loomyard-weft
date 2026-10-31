# Dobby lands a beam only once it is flagged ready with `lym beam ready`

A new beam with no work yet made its dobby ask the orch to run `lym beam land`, because the landing's refusal found nothing working under the beam and an empty beam read as a finished one.
A beam now carries a ready flag, and the dobby's landing step, `lym beam land` and the landing's re-check at Beam-Await all wait for it.

## What changes

- New verb `lym beam ready <slug>`, run by the beam's orch or the operator, sets the flag.
  It writes `beam-ready.json` (`slug`, `set_at`) in the beam worktree's `_lym/landing/` and commits it through `fabricengine.CommitAnchoredPaths`.
  A beam already flagged is reported as already ready; the verb only commits a record an earlier failed commit left behind.
  It refuses what `lym beam land`'s entry check refuses.
- `lym beam land` refuses an unflagged beam, after the owner checks and before the work-remaining refusal, and names `lym beam ready <slug>` as the way forward.
- Beam-Await's landing re-check adds a line for an unflagged beam, so a land request written past the verb cannot land one.
- The dobby no longer posts a "can land" notice for an idle beam.
  Its `CanLand` reading passes only for a flagged beam with nothing working under it, and no flag means no step, however long the beam has been idle.
  Once the reading passes, the dobby writes the land request itself, the one `lym beam land` writes, through the new `Deps.Land` seam.
  It posts to the beam's orch only when it cannot: another machine owns the parent (the notice names `lym adopt <subject>`), or `Land` refuses (the notice carries the refusal).
- `internal/dobbyengine/landnotice.go` and its test become `land.go` and `land_test.go`.
  The state record's `LandNotice` becomes `Land`, which also records whether the episode ended in a refusal.

## Design decisions

- The flag lives in the beam's tracked `_lym`, not on the board, so it survives a machine change.
  Weft content never merges into the parent, so a landed beam does not flag its parent.
  A pair forked from a flagged beam inherits the record, so a record naming another slug reads as no flag (`pawlengine.ReadyFor`).
  The paths are declared in `internal/loomengine` beside the landing directory; the record type and its write and read are in `internal/pawlengine`, through `internal/state`.
- The flag says the beam is ready to merge, not that the project is finished.
  The dobby never sets it, and no verb clears it: a beam flagged too early still lands only once nothing works under it.
- A refused landing stays an open episode, because the refusal may only be waiting for the run to reach Beam-Await.
  `Land` is asked again each tick, and the refusal is posted once.
  An episode closes when the land request is written, or when the notice about the parent's owner goes out.
  A reading that stops passing clears the episode, and a restarted dobby reads the record and does not act twice.
- The dobby lands nothing by itself: Beam-Await re-checks the beam before anything lands.
- `judgeLandEntry` takes the verb name, so the shared entry refusals open with `lym beam ready` or `lym beam land` as called.

## To check by hand

- A beam created before this change has no record, so its landing needs `lym beam ready <slug>` first.
- The flow on a real hub was not run: create a beam, confirm its dobby stays silent, run `lym beam ready`, and confirm the dobby writes the land request once nothing works under the beam.
