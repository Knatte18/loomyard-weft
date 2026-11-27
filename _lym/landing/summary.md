# Finalize gates the merged tree unlocked and locks the parent to land

`Finalize` took the parent lock before merging the parent into the task worktree and running the `landing` gate.
On a busy parent nearly every landing ran the whole gate under that lock, and every other landing on the parent waited.

`Finalize` now merges the parent's tip and gates the merged tree with no parent lock held.
It then takes the lock and lands if the parent's head is still the tip it merged.
If the parent moved, it frees the lock, merges the new tip and gates again, unlocked.
The cost is an extra gate run for each lost race.

The new key `landing.finalize_lost_races` (default 2) bounds the lost races in one `Finalize` call.
After that many, `Finalize` merges and gates under the lock, so a run is not starved.
`0` does so from the first pass, and a negative value is refused.
A hub whose config lacks the key gets the default from the template.

## Design decisions

- What lands is a tree that passed the gate on top of the parent it lands on.
  The locked pass still merges the parent and asks the gate; an up-to-date merge of an already verified tree skips the verify.
- The unlocked merge takes the parent's upstream tip.
  With the push skipped or no upstream, it takes the local tip, holding the lock only to read it.
  `ParentAlign` and `Finalize` share this choice as `parentMergeTip`.
- The owner checks run before the unlocked merge and again under the lock,
  so a run this machine may not land ends before anything is merged into its worktree.
- A parent that already holds the task's work gets no unlocked pass.
- After a lost race, the run waits for the lock at the front of the queue.
- A conflict session, merge fixer or fix review in the unlocked pass holds no lock.
  The locked pass resolves a conflict with the lock held, and frees it only for a merge fixer or fix review.
  The unlocked pass then runs again first, while lost races remain.
- The config-change notice now reads the task's config changes twice:
  the changes and their base before the first merge of the parent, which moves that base,
  and the parent's values again under the lock.

The recipes' comments, `docs/overview.md`, the landing-halts role text and the `landingshed` package doc describe the new order.

## Merge danger

**Door:** two-way
**Blast radius:** `landingshed`
