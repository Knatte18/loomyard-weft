# Audit: loom tail rework (PR-Rework, Merge-Review, verify retries) is rare and cheap

`pr-rework-audit` · Note · audit, loom, burler

## Seen (agent study, 2026-10-22)

Landed loom runs 2026-10-08 to 2026-10-22.

- PR-Rework: 8 of 130 runs (6.2%), never twice in a run, none in the last 27 runs (since 2026-10-16).
  Four rejects came from the orch or operator before PR-Align existed; four came from the alignment agent (5.6% of 72 verdicts), all valid.
- A rework cycle cost $2.94-8.97 extra (mean $4.82) and 15-42 min (mean 23); $38.58 in total, about 0.7% of loom spend.
  Each rework plan was one fix card; the second Plan-Review is the largest part.
- Where the deviation entered: decision record 1, plan 3 (Plan-Review missed it), code 3 (two of them written by the Webster-Review or Merge-Review fixer), not a real deviation 1.
- Merge-Review ran in 73% of runs since 2026-10-12, its fixer changed code in a third of those, about $0.96 per run.
- Verify failures at Publish/Merge-In: about 38, 29 retried automatically by batten.

## Idea (orch, not a decision)

The tail is not where the money goes.
If anything is done: a scope-alignment lens in Webster-Review and Merge-Review would catch the fixer-written deviations; a `small` reject verdict could route to a fix row instead of a new plan generation.
