---
round: 2
focus:
  - "In internal/shedadapters, `roundUnchanged` (bouncerstalemate.go) and `fixLeftTargetUnchanged` (bouncerverify.go) each guard on `record.TargetID == \"\"` so that a missing overlay target never reads as unchanged. For each guard separately: would a test in bouncer_stalemate_test.go or bouncer_verify_test.go fail if that guard were deleted, given a record whose TargetID and PostFixTargetID are both empty?"
  - "When an overlay target matches no file, `recordRoundStart` (burlerreview.go) logs a warning and writes the round's record with an empty target_id. Does a test in internal/shedadapters/roundsnapshot_test.go drive that failing snapshot and assert the written record?"
  - "`reviewTargetSnapshot` (entries_burler.go) sets the snapshot's WorktreeRoot to Env.AnchorPath. Do the doc comments on the TargetSnapshot fields in internal/shedadapters/targetsnapshot.go describe what the paths are relative to, also on a hub anchored to a subpath?"
  - "`reviewTargetSnapshot` serves both the review row (entries_burler.go) and the Bouncer (entries_bouncer.go), so the root change also reaches the round-start target_id and the round changes file written by `recordRoundChanges` (roundcontext.go). Does the `## Merge danger` section of _lym/landing/summary.md name a blast radius that covers both?"
---
