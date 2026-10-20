# Round 3 fixer report — discussion

Every finding's premise held against the code at HEAD (`boardCriticalSection`'s blocking `AcquireWriteLock`, `EndSession`'s refusal probe before the loop kill, `RemoveRefusal`'s force covering dirtiness only, the pause's `Added` skipped on `AlreadyPaused`, `PullThenCommitWritten`'s pull-first, `lockPairTeardown` and `runLockWait`).
All seven are fixed in `_lyx/discussion/decision-record.md`, and round 3 is logged in `_lyx/discussion/support-log.md` (Q26–Q29).

## Changes

- **F1** (fixed as proposed): D5 says the Bolt entry merge runs inside the caller's `board.lock` and `.gitrepo-push.lock`, takes neither, spawns no sync, and is a boardengine entry point beside `boardCriticalSection` that applies `validateWrite` and `checkRunLocks` over a Store loaded from origin's `board.json`.
  The unit of refusal is one entry or design-file change; a dependent later change is refused too; the refusal classes are listed (issue number added, since `validateWrite` checks it).
  The Bound and two Acceptance criteria were updated (mixed commit, no self-wait).
- **F2** (fixed as proposed, the force option): the busy-driver case passes force for dirtiness alone, a merge in progress still refuses, and once the driver is ended the local mode snapshots the uncommitted paths into a commit under a local `set-aside/` tag and reports them.
  The D8 notice, the Bound and the cleanup criterion cover the dirty worktree.
- **F3** (fixed differently, after self-check): the review's first option, exempting a live hub entry the request does not own, was carried out and then replaced by its second option.
  Passing an operator's pause contradicts D8's own rule that `lyx orch start` and `lyx cleanup` wait at the gate under it, and Q24.
  The wake reconcile is now skipped while a hub entry the request does not own is live, and the notice names that entry, `lyx resume` and the later reconcile; a criterion covers it, and D10's parked-push sentence names the operator's lift.
- **F4** (fixed as proposed, plus a guard): D3 step 2 reads removal from the fetched origin tip's own history for the slugs at issue; D5's and D10's reuses name the tip and its history.
  The self-check added a `created` match where local Bolt holds the slug, so a reused slug is never ended.
  A criterion covers a Bolt fast-forwarded past the removal.
- **F5** (fixed with the review's first option, after self-check): rewound needs origin's head to be an ancestor of an earlier tracking-ref value, and rewritten needs it to be an ancestor of none, so the two are disjoint.
  The first fix separated them by the unpushed commit instead, but a rewind with no unpushed commit then pointed at `lyx fabric pull`, which refuses it with `ErrWarpDivergedUnpushed` once the reconcile's fetch moved the ref (`internal/fabricengine/pull.go:255`, `:305`).
  The criteria have one case for each outcome.
- **F6** (fixed differently in part): D6 step 1 has the local mode try-acquire the teardown lock and the weft write lock and report busy when either is held.
  The review's proposal also asked for the loop and run locks to be try-acquired; that was not done, because those locks are held by the pair's own loop and step tree until the teardown's kill ends them (`seizeLoopLock`, `awaitPairRunLock`), and a try before the kill would read every paused pair as busy.
  Those two waits are kept, named and bounded by `runLockWait`; D4's Bound lists the teardown lock and the kept waits.
- **F7** (fixed as proposed, the backoff option): a gone push whose entry origin's Bolt shows neither done nor removed is classed as transport failure under the existing backoff; it records and mails only at the end of the backoff, and a later reconcile's D6 teardown removes a record that window still writes.
  The gone bullet's opening and a criterion were updated.

## Self-check

- fix-consistency, MEDIUM: rewritten had no precedence over ahead after the first F5 fix — fixed (by F5's final split, rewritten can never read as ahead, as D4 now says).
- fix-consistency, LOW: the rewound criterion after a fetch was left ambiguous — fixed.
- fix-consistency, LOW: D10's parked-push lift sentence was stale against D8 — fixed.
- fix-consistency, LOW: the gone bullet's opening still said a failed push at once — fixed.
- fix-claims, MEDIUM: try-acquiring the loop and run locks is wrong, because the pair's own loop holds them — fixed (see F6).
- fix-claims, LOW: D4's Bound named one kept wait where there are two — fixed.
- fix-claims, LOW: "the wake reconcile never waits at the gate" was too strong — fixed (the sentence was replaced by F3's skip).
- fix-proposal, MEDIUM: F5's split pointed a rewind at a `lyx fabric pull` that refuses it — fixed differently (ancestry split).
- fix-proposal, MEDIUM: F3's exemption broke an operator's hub pause — fixed differently (the wake reconcile is skipped).
- fix-proposal, MEDIUM: F4's history evidence ended a reused slug — fixed (`created` match).
- fix-proposal, LOW: the claim that force discards nothing missed ignored files — fixed (now named).
- fix-way-forward, MEDIUM: a second `lyx cleanup --apply` after a busy teardown lock met a dirty worktree without force — fixed (the case covers an entry its earlier run already ended, marked by the snapshot tag, and the snapshot runs before the teardown lock is tried).

## Disputed

None.

## Deferred

None.
