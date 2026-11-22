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
    disposition: fixed-differently
    reason: "A pair's pusher passes InsidePair true and exits, as proposed; the prime's pusher passes false but goes on with the other batten runs instead of exiting, since it serves every batten run (F2) and an exit would leave their commits to the next restart."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
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
---

# Round 1 fixer report — discussion

All changes are in `_lym/discussion/decision-record.md`, with a Round 1 entry in `_lym/discussion/support-log.md`.

## Changes

- F1: D5's first bullet names a `fabricengine.PairPush` mode that returns the owed `*PushFailedError` without writing a record.
  It applies at `recordOwedRecordsPush`, at `teardownLockOwedError` and at `pushClassified`'s owed-class failures.
  So `FailedPushesOwedBy` and the owed-push read in `runread.go` find nothing.
  The restart gate counts only refusal classes.
  D8 now rests on that mode, and D3's bound no longer says the pusher pushes only what `PushPairAnchored` already pushes.
- F2: D6's prime bullet works through a rejected push in this order:
  - It reads the takeover of every batten run that a `Run-Transition` trailer of an unpushed commit names, before any replay.
  - It hands each taken-over run over.
  - It archives that run's prime commits through a change to `HandOverTips`.
  - The replay then leaves those commits out.
  - The bullet names `retryWeftPush`'s `runSlug != ""` gate and its replay as the sites that change.
  - A new D6 criterion covers a batten run taken over while the prime holds its unpushed transitions.
- F3: D7 says that in the pusher a pause stops the push, where `skipsCheck` skipped the check, and that the pause is read before the Finalize skip.
  D3 and D5 now include "no pause covers its run" in the conditions for starting a pusher, and a D7 criterion covers a commit made after the pause.
- F4: D3 registers the pusher under `runproc.HubOwner` with role `runproc.RolePush`, and the D3 criterion names that key.
- F5 (fixed differently): D6 states `RunPush.InsidePair` true for a pair's pusher, which then exits, and false for the prime's pusher.
  I did not make the proposed "the pusher then exits" apply to the prime's pusher: it serves every batten run (F2), so it goes on with the runs nobody took over.
- F6: Scope In and D1 now cover `commitRunRecords` and the `commit-records` verb, which commit and start the pusher, and a D1 criterion covers them.
- F7: D6's bound changes the code-tip read so that it leaves `CodeTipMissing` unset when the run's pair stands on this machine and holds the tip.
  A read on another machine still sees the wait, covered by `CodeTipGrace` and the settle.
  A D6 criterion covers both cases.
- F8: D5 states that a refused prime push holds every batten run's records, accepts it with a reason, and adds the case to the refused-push criterion.
- F9: D5 has Worktree-Teardown push with no prime lock held, and its criterion says so.
  The self-check then refined the order (see below).
- F10: D1 now points to D5 as the one exception, D5's bound states the exception, and the second Auto-mode assumption is gone.
- F11: D4 is now a Constraint on `CommitRecordsPaths`, and its criterion is kept as a regression check.
  D1's bound cites only D3 and D6.
  D5 to D8 keep their numbers.
- F12: a branch with commits and no upstream counts as ahead.
- F13: the pusher holds neither the teardown lock nor the push lock while it sleeps, and keeps its pusher lock.
- F14: the refused-record condition is in D3's start rule and in its criterion.
- F15: the plan note drops "unused" and says that `lym cleanup`'s push spawn uses the variable.

## Self-check

- fix-consistency, LOW, D7 left open whether the `commit-records` verb still checks before it commits: fixed. D7 says the verb runs no check before its commit.
- fix-consistency, LOW, D3 and D5 keyed the refused-push gate differently: fixed. D3's start conditions name the records branch.
- fix-way-forward, MEDIUM, no next step after a failed takeover read: fixed. A failed read, archive push or hand-over counts as owed and the whole pass is retried; a D6 criterion covers it.
- fix-way-forward and fix-claims, MEDIUM, merged: the prime-commit archive and the replay filter had no site: fixed.
  D6 names the `HandOverTips` change and `retryWeftPush`'s replay.
- fix-way-forward, MEDIUM, a failing hand-over of one run left the others with no way forward: fixed. It counts as owed and the whole pass is retried.
- fix-way-forward, LOW, a handed-over pair's pusher restarted on every tick: fixed. A hand-over mark is now one of D3's start conditions, and the D6 criterion covers it.
- fix-proposal, MEDIUM, with no deferred record, `recordsSideNotLanded` cannot tell a deferral: fixed. In the mode it reads the deferral from `pushClassified`'s returned error.
- fix-proposal, MEDIUM, a records branch pruned after origin ended it read as ahead forever: fixed.
  A branch counts as ahead only when it has no upstream; the pusher sets the upstream on its first push, so a pruned branch reads as level.
- fix-proposal, MEDIUM, a push before the session shut down missed its last commits: fixed.
  Worktree-Teardown shuts the session down, releases the prime lock and pushes.
  It then re-takes the lock and removes the pair only while origin's tracking refs hold the tips.
- fix-proposal, LOW, nothing said how a pusher exits once its worktree is removed: fixed. A pusher whose worktree is gone exits, with a D3 criterion.
- fix-proposal, LOW, an owed backoff was invisible without a record: fixed. The pusher posts one creel notice once its backoff reaches 60 s.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
