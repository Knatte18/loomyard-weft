# A landing queue gates several ready runs as one batch and lands them together

`landing-queue` · Note · enhancement, landing, gate, batten

## Seen (operator, 2026-11-27)

Each landing ran its own landing gate under the parent lock, one after the other, so with many runs ready the gates queued in series.

## Since (`d87a9ce24`, `landing-lock-only-for-land`)

`Finalize` now merges the parent's tip and gates the merged tree with no parent lock held, and takes the lock only to land.
Ready runs therefore gate side by side.
A run whose parent moved during its gate merges the new tip and gates again,
and after `landing.finalize_lost_races` lost races (default 2) it merges and gates under the lock.
Each ready run still pays for its own gate, plus one for each race it loses.

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
