---
round: 1
focus:
  - "Cards 18, 41, 42, 43, 44, 45 and 23 (Delete groups): confirm each deleted symbol passes assert-no-callers, with no leftover reference in code, stencils, recipes or config."
  - "Cards 11, 26, 29: the fixer loop shared by Merge-In and Finalize; check that the attempt record, cap judgement and halt destinations match the plan on every exit, including a released parent lock."
  - "Cards 27 and 30: Finalize takes the lock first and releases it on every exit path, including error and reroute paths."
  - "Cards 34 to 38 and 40: the hand-back mark is honoured by the Go driver, arbiter, batten re-step and LLM driver stencils alike; check that no consumer is missed."
  - "Cards 21, 47, 48, 49 and 50: config and run-settings migrations; check that old runs and old keys still read, and that stencil wording changes drop no rule the card's Intent did not ask to change."
  - "Cards 20 and 22 and the last commit's fixture rows: recipe tails follow the plan's row order and the PR-Review stub flag, and the commit message of any departure carries a valid Pattern-Departure trailer."
  - "Card 52 and the declared PATTERN changes: each declared entry lands with its declared text."
  - "Test economy across the new fixer, hand-back and migration tests: new tests that duplicate an existing covering test, and added behavior with no coverage."
---

Seed focus drawn from the plan's card index; the range is the branch's commits over the merge base with the parent.
