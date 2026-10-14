---
round: 1
focus:
  - "Cards 4-6: does the darn discussion gate commit the record at the writer's first clean validation, and does a loom build (no `Env.DiscussionHistory`) resolve and validate exactly as before?"
  - "Cards 2-3: does the kept-entries check compare against every committed version of the record through `fabricengine`, and refuse a record that dropped an `Added after Discussion` entry?"
  - "Card 7: does `decision add` on a darn run append only once the record is committed, and do the new way-forward messages match the behavior?"
  - "Cards 10-13: is the PR-Align row wired with darn's geometry only on a darn run, and does the told pr-gate measure leave the loom path unchanged when no plan exists (`none`)?"
  - "Per-card mechanical check: for each card's groups, did the type-specific check run against that group's own targets?"
  - "Test economy: do new tests duplicate existing coverage, or are behaviors added without any?"
---

Focus is where to look first; the whole committed diff stays in scope.
