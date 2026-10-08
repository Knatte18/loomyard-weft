---
round: 1
focus:
  - "Cards 1–3 (`discussionparser.WriteCarryOver`, the Bouncer carry-over seam, `carry_over` key and `Env.CarryOver`): does discussionparser alone render, sanitize and splice the entry, with the Bouncer passing structured `CarryOver` data only, and does the carry-over run before Approve on every judged Done?"
  - "Card 2 and card 9: do the Bouncer's Stuck-path refusals and the carry-over refusal keep `lyx loom start`, with `circlingWayMalformed` as the one stated exception, while only the awaiting-Bouncer ways forward move to `lyx loom resume`?"
  - "Card 5 (round-2 convergence rule, earlier-open facts list, judge stencil): does the rule converge only when the latest round has no BLOCKING finding and no MEDIUM-or-worse `design` finding on a key already open earlier, and does it leave round 1 unchanged?"
  - "Cards 6–8 (template budget of three, six per-segment model keys, `fix_start`): are the new `loom.yaml` keys parsed, validated and defaulted consistently, and does `Env.RowReviewModels` reach BurlerRound rows keyed by row name?"
  - "Per-card mechanical checks: for each card's groups, did the type-specific check (Rename script plus grep, `assert-no-callers` for Delete) run against every group's own targets?"
  - "Test economy: do new tests duplicate coverage from existing ones, and does each added behavior have coverage?"
---

Focus covers the plan's riskiest seams: the sole-parser boundary, the refusal routing split, and the convergence rule's edge at round 2.
