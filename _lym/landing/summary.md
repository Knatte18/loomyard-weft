# Worktree-Create holds the prime lock only for the hub's registry writes

Worktree-Create held the hub's prime lock for the whole create closure.
That covered the board claim's Bolt pull, commit and push, fabric `add`'s two `ls-remote` probes, the config reconcile and the new pair's push spawn.
Tasks released together therefore left the row one at a time.
On 2026-11-04 five runs took 56 s, about 3 min, about 4 min and 6.5 min, each waiting for the one before it to finish its network calls.
GitHub was degraded at the time, so every network call was slow.

## What changes

- `battenshed`'s Worktree-Create producer no longer takes the prime lock around the closure.
  It hands the closure a `HoldPrimeLock` instead, which runs the same bounded wait as before (every 2 s, up to 10 min).
  A wait that runs out stands the row Stuck, a lock failure is a hard error, and a cancel during the wait is the cancelled-during-run error, as before.
  The row releases any hold the closure leaves taken, and a release runs once however often it is called.
- `shedrecipe.Env.CreateWorktree` takes the hold as a second argument.
- `fabricengine.AddOptions` gains `Hold`, a general lock seam that names nothing of batten.
  `Add` reads whether the two branches exist and starts both origin probes in the background.
  Only then does it take the hold, and it keeps the hold to its return, its rollback and the workspace-file regeneration included.
  So the source-dirt check, the fork point's marker, the HEAD read and every worktree and branch write run under the caller's lock, but the probes' network wait does not.
  Every return waits for the probes, and the pre-flight refusals keep their order: a probe's answer is read only where it was read before.
- The battencli closure passes the hold to `Add`.
  It also takes the hold in `refuseHeldMember`, and only around the removal of a held beam member's pair, since that writes the registry.
  The board claim, the beam fork-parent read, the complete-pair skip, `reconcileTaskPair` (which takes the hub reconcile lock of its own) and `spawnPairPush` all run unlocked.

## Design decisions

- The board claim's Bolt push stays synchronous.
  It is the commit that tells another machine this one owns the entry (PATTERN-owner-machine).
  Pushing it in the background would let two machines start the same task.
  It now delays only its own run.
- The hold is taken inside `Add`, after the probes start, rather than around `Add`.
  The probes are the network part of `Add`, and the pre-flight reads must stay under the lock: Land pulls the prime under the same lock, and a dirt check that read the prime mid-pull could halt a create on a false `ErrSourceDirty`.
- Lock order inside `Add` is the slug's teardown lock, then the prime lock.
  Worktree-Teardown takes the prime lock and then the teardown lock, but only for its own slug, which never creates and tears down at once, so the two orders cannot deadlock.
  The origin reconcile only try-acquires the prime lock.

## Known limits

- One more queue remains: the weft repository's remote-refs lock.
  The Bolt checkout is a worktree of the weft repository, and a push holds that lock for its whole transfer.
  So while GitHub is slow, every board claim still waits behind any weft push in flight.
  Changing that touches PATTERN-remote-refs-lock's judges, so it is filed as board note `push-holds-remote-refs` rather than changed here.
- Why Open-Window took 69 s in the incident is not established: the step left no Info-level trace.
  The same note records what is known.
- A create still makes two Bolt round trips, the start check's pull and the claim's pull and push.
  This change does not merge them.

## Tests

- `battenshed`: the existing producer tests now take the lock through the closure's hold.
  A new test pins that a closure taking no hold never acquires the lock, even while it is contended, and that a hold released twice releases once.
  The lock-wait-ran-out case also asserts it stands Stuck without a halt.
- `fabricengine`: `TestAdd_HoldsTheCallersLock` pins that `Add` takes the hold once and releases it by its return around a pair it makes, and that a failed hold is returned as is with nothing made.
- `battencli`: `TestRefuseHeldMember` pins that a member that is not removed takes no prime lock.
  The integration tests call the closure with a hold that takes nothing.
