---
round: 1
focus:
  - "Cards 1–2: does the shared admit check in the arbiter and tackler consumes admit only a verdict already in effect, leave the refusal texts unchanged, and have a relaunch skip its goto when Admit reports the target in effect?"
  - "Card 3: does the arbiter refuse a Merge-In or Publish verify halt only when the halt's own failure record names a target, and still act in every other halt case?"
  - "Card 4: do the `row` and `own_row` fields of `lyx loom peers` carry each run's embedded-recipe row and the caller's own, and does the alignment stencil's landing-order rule read them correctly?"
  - "Cards 6–8: does the digest's `deviations` come from the scope guard alone, file-granular, with `Report.Deviations` kept as an ignored field and the JSON name and `omitempty` unchanged, and does it reach a reader once?"
  - "Per-card mechanical checks: for each card, did every group's type-specific check run against that group's own targets?"
  - "Test economy: do the new tests duplicate an existing test's coverage, or leave an added behavior uncovered?"
---

Seed focus from the plan's card index and shared decisions; the reviewer judges the committed diff on its own as well.
