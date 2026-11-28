---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The proposed hold-off through a pair push lock the leader takes rests on a lock that does not exist as proposed: the only pusher lock is the pusher process's own single-instance lock (`internal/runpusher/start.go:213`, `tryLock`), held by the running pusher, so a leader cannot take it to hold a live pusher off. D5 instead has the pusher and `RefusedPushStands` hold off the code side while the batch's scratch marker (F10) stands. The other readers are named as proposed."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
---

## Changes

Files changed: `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`.

- F1: D6 has the leader judge, for each member with its own seams, `ownerChecks`' reconcile verdict and owner read, `landCheck` and `handedOverStop`'s owner read; a failing member is left out with no outcome and the chain is rebuilt and gated again.
  Constraints gain, under `PATTERN-owner-machine`, that a member's ownership is judged under the lock before the move.
  Criterion added.
  The self-check made these judgements reads alone (see Self-check).
- F2: D1's pre-join read is now `needsGitHubOnDisk` alone; D6 runs the pull-request resolver per member under the lock; D10 says the decision is made under the lock for every run and keeps its GitHub-rules bound.
  D2 names what the pair path lets the leader build: the member's reconcile and owner reads, beam land check, pull-request record path and lock, beam landing record, beam mode, origin and task-has-commits read.
  I read them in `finalize_pr.go:257-315` and `finalize.go:619-661`, and each comes from the run's own worktree, so the pair path suffices.
  Criterion added.
- F3: D8 opens with the `ParentHoldsTaskWork` read before any outcome; D3's leader excludes a run whose work the parent holds; D4 leaves out an entry the previous chain commit already holds (`Fabric.HoldsWork`); the backstop sets `landed`; Constraints say the queue is a cache.
  Criterion added.
- F4: D6 writes a move mark before the move and removes it after every ledger landing and outcome; the backstop reads the mark before any pruning; D2 orders the pruning after the backstop.
  Criterion added.
  The self-check changed the non-run lock taker's part (see Self-check).
- F5: D7 gives a culprit `failed` whether or not it is the leader. Criterion added.
- F6: D8 splits `conflict` from `failed`; a `conflict` run frees the lock and merges with no lock held, then lands alone from the front of the queue. Criterion added.
- F7: D6 re-reads each member's pause cover, liveness and task tip right before the move. Criterion added.
- F8: D2 says Done removes the entry and a halt or Stuck keeps it, and how a rejoin rewrites it; D8's `landed` bullet needs the task tip equal to the entry's task head. Criterion added.
- F9 (fixed differently): D5 names each reader of the detached worktree and what it does, and its Bound gains the darn tree-scope point. Criterion added.
  The pusher hold-off is through the batch marker rather than a pair push lock; the reason is in the frontmatter.
- F10: D5's restore moves into the run's resume, guarded by a scratch marker; a detached worktree without the marker is Stuck. D5's criterion now names a resumed run at any row.
- F11: D5 removes leftovers through `Deps.RemoveLeftovers` after each checkout and the restore. Criterion added.
- F12: D6's Bound says the hold starts at the ledger write; the records commit is picked as `oldestHeldRecords` picks it today, with the join-time records head as fallback (D2).
- F13: criteria added for D5's dirty-detached Stuck, D7's leader-culprit path, D2's entry removal at Done and D6's records hold.
- F14: D4 logs batch-gate flaky failures at Warn with the chain commit; D7 runs the batch gate with no failure recorder (the self-check corrected the flaky-callback part).
- F15: D7 names landingshed's rerun of a test-run failure, and its Bound covers ungated upstream commits, timeouts and build failures.
- F16: D7 sends a leader-culprit to its fixer on the prefix-1 log; D6 bounds rebuilds at `landing.batch_max` per lock hold; D9's rationale is per chain built; support-log Q7 says two.
- F17: D8's second bullet and the D6, D8 criterion name a parent reset outside the queue.
- F18: D9 has the key join landing's `RetiredKeys` as `CategoryCrossPair`, and its criterion names the unknown-key warning.

## Self-check

- fix-consistency, MEDIUM, D2's rejoin rule drops `failed` after a fixer commit: fixed. D2 keeps `failed` and `conflict` until the run lands alone.
- fix-consistency, LOW, D10's title and criterion against its new body: fixed. Title and criterion reworded.
- fix-consistency, LOW, D7's failure-record sentence against the leader-culprit path: fixed. The sentence names the exception.
- fix-claims, MEDIUM, a nil flaky callback runs no rerun (`verifygate.go:47-49`): fixed. The batch gate has a flaky callback of its own that logs at Warn and leaves the leader's flaky list alone.
- fix-claims, LOW, "which the rerun does not repeat" for failures never rerun: fixed. Now "does not run again".
- fix-proposal, MEDIUM, the member checks act (hand-over, follow-up removal, fast-forward, pull-request close): fixed. D6 judges them as reads alone, and a member whose check needs an action is left out.
- fix-proposal and fix-way-forward, MEDIUM, merged: the marker names one commit, but bisection checks out several: fixed. The marker names the task branch and stands across the lock hold; the restore acts whatever the detached commit.
- fix-proposal, MEDIUM, a leader-culprit's fixer gets a log from a tree its worktree lacks: fixed. The leader first merges the chain base into its worktree under the lock.
- fix-proposal and fix-way-forward, LOW and MEDIUM, merged with the consistency rejoin finding above: fixed by the same D2 change.
- fix-way-forward, MEDIUM, the no-marker Stuck names no way forward: fixed. Both D5 Stucks name theirs.
- fix-way-forward, MEDIUM, a standing mark after `lym fabric pull` is cleared by no Finalize when none is queued: fixed. Every parent lock taker, fabric verbs included, runs the backstop; this replaces F4's proposed "leaves a standing mark alone".
- fix-way-forward, LOW, the already-landed path writes an owed landing for a pushed tip: fixed. Both D8 writes happen only while the upstream lacks the landing commit; the backstop has the same rule.
- fix-way-forward, LOW, a `conflict` run's merge under the lock can still need a conflict session: fixed. Such a merge frees the lock and runs the lock-free merge again first.

## Disputed

None.

## Deferred

None.

## Rulings

None.
