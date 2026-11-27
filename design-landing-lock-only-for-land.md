# Finalize gates the merged tree without the parent lock and holds the lock only to land

`landing-lock-only-for-land` · Task · enhancement, landing, gate

Depends on: [`transient-stop-not-failed`](design-transient-stop-not-failed.md)

## Seen (operator, 2026-11-27)

Finalize takes the parent lock, merges the parent into the task worktree, runs the `landing` gate and only then lands.
The gate is skipped only when the parent has not moved since the tree was last verified.
With many runs in flight the parent moves between almost every landing,
so nearly every landing runs the whole landing gate under the lock, and every other landing on that parent waits for it.
Under today's load a landing gate takes 4 to 12 minutes.

## Idea

Run the merge and the landing gate with no lock held, and take the parent lock only to land:

- Merge the parent's tip into the task worktree and gate the merged tree, unlocked.
- Take the parent lock and check that the parent's tip is still the commit the gated tree was merged from.
- If it is, land at once; the lock is held for seconds.
- If the parent moved, release the lock, merge the new tip and gate again, unlocked.
- After a bounded number of lost races, fall back to today's path, merging and gating under the lock, so a run is never starved.

No drift is possible: what lands is always the tree that passed the gate on top of the parent it lands on.
The cost is an extra gate run when a run loses the race.

The fixer and fix review already run with the lock freed; the change is to the merge and the gate around them.
