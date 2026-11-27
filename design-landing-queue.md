# A landing queue gates several ready runs as one batch and lands them together

`landing-queue` · Note · enhancement, landing, gate, batten

## Seen (operator, 2026-11-27)

Each landing runs its own landing gate, one after the other, so with many runs ready the gates queue in series.

## Idea

A landing queue in the manner of GitHub's merge queue, bors or Zuul:

- A run ready to land joins a queue for its parent branch instead of taking the parent lock.
- The queue takes the waiting runs, say A, B and C, builds the tree parent + A + B + C and runs the landing gate once on it.
- A pass lands all of them together.
- A failure is bisected (parent + A + B, and so on): the runs that pass land, and the culprit goes back to its own run's fixer.
- A later step may gate the prefixes in parallel (parent + A, parent + A + B, parent + A + B + C) so a failure costs little time.

No drift: the batch is gated exactly as it lands.
It saves the most gate time when many runs land often, and it costs the most to build:
a queue service, batch merging, bisection on failure and fixer handling for the culprit.
It builds on the unlocked gating of the other note, which comes first.
