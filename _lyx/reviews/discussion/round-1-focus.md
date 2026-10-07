---
round: 1
focus:
  - "Decision 1 against Decisions 3 and 4: the calibration report runs the shared estimator, which lives in `internal/batcher`; settle whether 'calibration first in the card order' is satisfiable as written, or whether the estimator must precede the report."
  - "Decisions 6 and 7: a single card whose estimate is below `alone_above` but above `budget` has no split that meets the hard limits; settle whether the record states what the DP does then, and what bounds it."
  - "Decisions 4 and 5: Decision 4 says a card yields an estimated message count and an added context, Decision 5 prices a fork from read sets and a start-up context; settle whether the two describe one model, or whether a quantity the plan writer needs is defined in only one of them."
  - "Decision 3, the Auto-mode assumptions and Decision 2: three phrasings pick the plan and the base tree for a past run (last plan-artifacts commit before the first fork, tree the first batch started from, tree before the first card commit); settle whether they name one rule."
  - "Decisions 10, 11 and 12: where the order assertion runs once the partition is recorded, and what a recorded partition does when the current plan has gained, lost or reordered cards outside a rebaseline; settle whether each case has a stated bound or refusal."
  - "Decision 11 and Decision 8: the rebaseline re-batch and the profile load error can both fire mid-run; settle whether the record bounds what a mid-run load error does to an already recorded partition."
  - "Relocation pass: check whether Decision rationales or the Notes for the plan writer carry deliberation that belongs in `support-log.md`, applying the completeness-before-leanness test before any relocation finding."
---

Seed focus for round 1.
No entry targets the three `Do not flag` items or anything the discussion stencil told the writer not to gather.
Entries name where to look and leave severity and verdict to the reviewer.
