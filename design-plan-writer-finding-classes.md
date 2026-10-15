# Classify Plan lens and review findings to make the Plan writer get it right first

`plan-writer-finding-classes` · Note · audit, loom, burler, stencil

## Seen (2026-10-15)

Over the 15 most recent full loom runs (`tools/tokencount/tokencount.sh -last 15`), Plan cost about $13 a run: the plan lead about $5.50, its forks (fork0 and the five lens forks) about $4, and Plan-Burler-Review about $3.80.
Each fix the lenses or the review ask for is work the writer did not get right the first time.

## Operator view, not a decision (2026-10-15)

Plan is heavy by nature, since it turns the Discussion into concrete work; the aim is a smarter Plan writer, so fewer Burler rounds are needed.

## Idea (orch recommendation)

- A fresh agent classifies every finding the Plan lenses and Plan-Burler-Review raised in recent runs: what kind of defect, whether it was real, and whether the review found something the lenses had not.
- Each recurring class goes where it is cheapest to stop at the source:
  - a machine check in `lyx loom validate-plan`, when the class can be checked deterministically;
  - a writer rule in the Plan stencil, when it is a judgement the writer can make up front;
  - a missing input, when the writer lacked something it needed (for example what `discussion-relevant-list` now hands it).
- Then measure whether lens and review findings per run drop.
