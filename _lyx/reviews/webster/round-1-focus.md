---
round: 1
focus:
  - "Card 7 and card 13: does the seed-commit drop run only behind the destruction gate, and does the single retry after a rebase conflict leave the board clean when the retry also fails?"
  - "Cards 3 and 4: does the forward-only refresh rule order by build ancestry or commit time as the banner records, and does it refuse a dirty or unstamped binary as the plan states?"
  - "Cards 7, 8, 10 and 11: is each skip Warn logged once inside `Bolt.pullFirst`, and does every caller write nothing when the pull cannot bring Bolt up to date?"
  - "Card 11: does `lyx fabric reconcile` fail on both a hub-config and a warp-binding commit or push failure, with the failure reaching the verb's exit status?"
  - "Card 12: does gc at pair removal skip under another pair's live reed session, and does the day-old loose-object prune leave newer objects untouched?"
  - "Test economy: do the new tests duplicate coverage already given by a public-surface test, and does every behavior the diff adds have coverage?"
---

Focus is where to look first.
The rest of the rubric still applies to the whole committed diff.
