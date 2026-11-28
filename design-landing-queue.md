# A landing queue gates several ready runs as one batch and lands them together

`landing-queue` · Task · sketch, landing, gate, batten · starting · Worktree-Create

Decision record: [Sketch](decision-landing-queue.md)

## Seen (operator, 2026-11-27)

Each landing ran its own landing gate under the parent lock, one after the other, so with many runs ready the gates queued in series.

## Seen again (2026-11-28)

With five runs ready at Finalize, each landing made every waiting run's lock-free gate stale:
the parent moved by real code, so each run in turn merged the new tip and gated tiers 0 to 2 again under the parent lock, 10 to 15 minutes each,
while the waiting runs' own lock-free gates competed with the holder for the same gate slots.
The operator released this entry ahead of the measurement of `landing-lock-only-for-land` on that ground (operator, 2026-11-28).

## Since (`d87a9ce24`, `landing-lock-only-for-land`)

`Finalize` now merges the parent's tip and gates the merged tree with no parent lock held, and takes the lock only to land.
Ready runs therefore gate side by side.
A run whose parent moved during its gate merges the new tip and gates again,
and after `landing.finalize_lost_races` lost races (default 2) it merges and gates under the lock.
Each ready run still pays for its own gate, plus one for each race it loses.

Since `5c985d926` (`finalize-owed-tips-gate`), this also holds when the parent is ahead of origin only by owed landing tips,
except for a landing that needs GitHub only because of GitHub's repository rules.
Before, every landing on such a parent lost all its races and gated under the lock.

## Idea

A landing queue in the manner of GitHub's merge queue, bors or Zuul:

- A run ready to land joins a queue for its parent branch instead of gating and taking the parent lock itself.
- The queue takes the waiting runs, say A, B and C, builds the tree parent + A + B + C and runs the landing gate once on it.
- A pass lands all of them together.
- A failure is bisected (parent + A + B, and so on): the runs that pass land, and the culprit goes back to its own run's fixer.
- A later step may gate the prefixes in parallel (parent + A, parent + A + B, parent + A + B + C) so a failure costs little time.

No drift: the batch is gated exactly as it lands.
It saves the most gate time when many runs land often, and it costs the most to build:
a queue service, batch merging, bisection on failure and fixer handling for the culprit.
