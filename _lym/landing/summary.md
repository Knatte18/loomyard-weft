# Hash the Bouncer's review target under the anchor, refuse an empty one

The Bouncer's stalemate check compares a round's review target id before and after the round's fix.
For an overlay target, such as a Discussion segment's decision record and support log,
that id is a SHA-256 over the target files.
The snapshot resolved the target paths against the worktree root,
while the round's reviewers and fixer resolve the same paths against the anchor path.
On a hub anchored to a subpath of its worktree, the snapshot therefore missed the files those agents edit
and hashed empty input (`e3b0c442…b855`).
A round whose fixer had edited the decision record then escalated with cause `stalemate`,
and its changes file listed nothing.

Closes #287

## Changes

- `internal/shedrecipe/entries_burler.go`: `reviewTargetSnapshot` roots the overlay snapshot at `Env.AnchorPath`.
  The review row and the Bouncer both build their snapshot through it,
  so the round-start id, the changes file and the post-fix id now read the files the agents edit.
- `internal/shedadapters/targetsnapshot.go`: an overlay target with no file at all is an error
  instead of an id over empty input.
  The round's record then holds no target id, and a round with no target id counts as changed,
  so a missing target cannot produce a stalemate.
  A missing path beside an existing file is still skipped.
- Comments that called the target paths worktree-relative no longer do.

## Design decisions

- The overlay snapshot keeps its `WorktreeRoot` field name.
  It means the root the target paths resolve against,
  as the same field of burler's geometry does, which also holds the anchor path.
- An empty target fails the id read, not the round.
  The review row and the Bouncer log the failure at Warn and go on, as they do for any failed id read,
  so a misconfigured target does not fail the run.
  It does cost the stalemate check, and the following review round gets no changes file:
  that round is not told what changed, and its new findings are held to BLOCKING alone.

## Tests

- `OverlayTargetRecordsTheRoundSnapshot` in `TestBurlerReviewEntry_ProfileMapping`
  now puts the target under the anchor alone.
- `TestTargetSnapshot` checks that `ID` and `Take` both fail when no target path exists.
- `TestBurlerReviewProducer_Call_RecordsRoundSnapshot` checks that such a target leaves an empty target id
  in the round's record.
- `TestBouncer_Stalemate` and `TestBouncer_FixVerification` gain rows for a round with no recorded target ids.

## To check by hand

- The empty-target-id guard in `roundUnchanged` (`internal/shedadapters/bouncerstalemate.go`)
  alone keeps a missing overlay target from reading as a stalemate, and it had no test before this change.
  Check that the added row `RoundWithNoTargetIdCountsAsChanged` in `bouncer_stalemate_test.go` pins it.
- `fixLeftTargetUnchanged` (`internal/shedadapters/bouncerverify.go`) has its own empty-id guard,
  which decides whether a fix is verified and has no part in the stalemate check.
  This change adds `ARecordWithNoTargetIdsIsVerified` in `bouncer_verify_test.go` to pin it;
  check that it does.

## Merge danger

**Door:** two-way
**Blast radius:** review
