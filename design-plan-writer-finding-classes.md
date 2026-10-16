# Classify Plan lens and review findings to make the Plan writer get it right first

`plan-writer-finding-classes` · Note · audit, loom, burler, stencil, shed

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

## Operator view, not a decision (2026-10-16)

The lens fan and twill fixer (`fan-prompt-tuning`) are built first.
Later, this audit runs over real runs made with them, and the wording of the reviewer and fixer prompts is adjusted from what it observes, beside the Plan writer measures above.

# Moved from note `perch-review-analysis` (issue #676, 2026-10-16)

The analysis measured rounds, tokens and late BLOCKING findings per perch; its classification of the late BLOCKING findings moved to `fan-prompt-tuning`, the lens fan and twill fixer it proposed are built there, and its proposal of two parallel round-1 reviewers is replaced by the lens fan.
Its Webster proposal (a hard cap of 2) is already met by the hub's `loom.yaml`: `webster_max_bounces: 1` with `webster_auto_accept: true`.
Two of its proposals are left for this audit to weigh, numbered as in the original:

3. **Later rounds review the fix delta.**
   A findings cannot be caught in round 1; they need a round after the fix.
   From round 2, one reviewer checks only what the fixer changed plus the sibling decisions or cards it touches, which is cheaper than a full review.
   Compare Traycer's incremental re-verify in #671.
6. **Measure again in a week**, with the same scripts: rounds per perch, BLOCKING share in round 1, and tokens per perch, before and after twill and the changes above.
