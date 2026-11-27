# Finalize merges and gates unlocked, holding the parent lock only to land

Finalize used to take the parent lock first, then merge the parent into the task worktree and run the landing gate under it.
The gate skips only a tree it already verified, and on a busy parent the parent moves between almost every landing.
So nearly every landing ran its whole gate under the lock, and every other landing on that parent waited for it.

Finalize now runs the merge and the gate before it takes the lock:

- With no lock held, it merges the parent's tip into the task worktree and runs the `finalize` gate key over the merged tree.
  The fixer and the fix review that a failing gate calls for run unlocked as well.
- It then takes the parent lock and runs the existing owner checks and parent sync.
  It compares the parent pair's head with the tip it merged.
- If they match, the locked merge is already up to date and the verified tree skips the gate,
  so the lock covers only the checks and the landing.
- If the parent moved, Finalize frees the lock, merges the new tip and gates again with no lock held.
- After `landing.finalize_lost_races` lost races in one call, Finalize merges and gates under the lock, as it did before,
  so a run on a busy parent is never starved.

What lands is always a tree that passed the gate on top of the parent it lands on.
A run that loses the race pays one extra gate run.

## Design decisions

- The lock-free pass picks its tip the way Parent-Align does, through `parentMergeTip`, now shared by both rows.
  With an upstream and no push skip it fetches and merges the upstream tip,
  which the locked sync then fast-forwards the parent pair to.
  With the push skipped or no upstream it takes the lock only long enough to read the local tip.
- A parent that already holds the task's work runs no lock-free pass.
  That path skips the sync, so it keeps today's locked merge and gate.
- `landing.finalize_lost_races` is a new hub-wide key, default 2, in the cross-pair category like the other parent-lock keys.
  0 merges and gates under the lock from the first pass, which is the old behaviour, and a value below 0 is refused at load.
- `freeLock` now frees only a lock this process holds,
  so a fixer or fix review that starts during the lock-free pass releases nothing.
- After a lost race the next locked pass queues at the front, as a pass that follows a fixer already does.

## Tests

`TestFinalize_GatesWithNoLockAndLocksToLand` drives Finalize over a real lock store.
It records whether the lock was held at each verify and which tip each merge of the parent used. It covers:

- an unmoved parent;
- a parent that moves during the lock-free gate;
- a spent lost-race bound;
- a bound of 0;
- a parent that already holds the work.

`TestLoadConfig_InvalidKeys` gains a row for a negative bound.

## Merge danger

**Door:** two-way
**Blast radius:** landing
