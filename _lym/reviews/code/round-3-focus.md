---
round: 3
focus:
  - "internal/shedadapters/bouncer.go, Call and settleUnlessRefused: does every path that settles a judge verdict (live-judge attach, entryHarvest, recorded-verdict replay, fresh harvest) refuse a CONVERGED verdict when the round's parent ask holds findings, and is that check in one place?"
  - "internal/shedadapters/parentseat.go and contracts/stencils/bouncer: is the Bouncer judge's parent-review instruction text read at call time through stencilstore from a stencil fragment for the judge's engine, with no embedded Go constant, per PATTERN-stencil-ownership?"
  - "internal/driftengine/verify.go, judge, passCommand and runAgent: is the judge's start time set on both the line-mode and the diff-command path before the parent agreement entry is built, so the parent ask's deadline is capped by the judge run's timeout?"
---

The focus covers the three fixes this round's review required.
