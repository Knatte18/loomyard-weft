---
round: 1
focus:
  - "Cards 14 and 18: does the diff's cancel order follow the plan's D2 to D9 steps, with the run-lock wait after the owned processes and strands are ended, and the board claim cleared by the cancel rather than the teardown removal?"
  - "Card 19 and the `--remove` path: does the removal snapshot uncommitted paths and archive then delete task branches, locally and on origin, only after the checks the plan puts first, and do the production seams match what `cancelRun` expects?"
  - "Cards 16 and 17: do the start guard and the pre-drive check refuse a marked run, a dirty source and a kept pair's left parts before anything is written, and does nil-seam behaviour keep the existing untagged tests unchanged?"
  - "Cards 8 to 10: does Worktree-Create halt with the precondition reason the driver hands back for a dirty source or a left task branch, and is the typed error mapped without string matching?"
  - "Card 14 and the PATTERN change: does `PATTERN-pair-teardown` in the diff carry the declared changed text?"
  - "Test economy: for each new test, does an existing test already cover the behavior, and does each added behavior (cancel marker liveness lock, Windows process creation time, task-branch discard) have coverage?"
---

Seed focus drawn from the plan's card index and shared decisions; the reviewer verifies each against the committed diff.
