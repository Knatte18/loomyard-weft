---
round: 2
focus:
  - "In contracts/stencils/loom/loom-rubric-raddle-review.md, read the `design` class sentence and the concurrency judgement bullet together: does the sentence fault a chain that the work forces, and does it name a part added only so parts run at once? Settle whether the two agree."
  - "In the same file, read the concurrency judgement: does its test count both a short `depends_on` chain and the parts with no `depends_on` path between them, so that two cuts with the same part count but different parallelism are told apart? The different-packages half belongs to the `raddle` gate's `packages-shared` check and is not this file's to re-derive."
---

Both entries target the wording changed by the fixes to the round's design findings.
