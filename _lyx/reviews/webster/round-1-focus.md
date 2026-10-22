---
round: 1
focus:
  - "PATTERN.md and pattern/ diff: does it land each declared entry (`PATTERN-run-process-owner` add, `PATTERN-pair-teardown` change, `PATTERN-leaf-packages` change) with the declared text?"
  - "internal/runproc and every verb that ends or tears down a run (pair teardown, `lyx fabric remove`, `lyx reed down`, owned-elsewhere pauses, `lyx tender reap`): does each end processes only through `runproc.End`, called with the caller's own pid so batten and its ancestors are spared?"
  - "Detached pushes and board syncs (cards 3, 6, 7, 8): is every spawn registered under its owner, does a deferred child keep flags and commits, and does every lift spawn what a pause deferred, including a board sync for uncommitted board changes?"
  - "Teardown ordering (cards 9, 10): are owned processes ended before the loop and session, and does the run-lock wait precede the owned-process end while the task worktree is present?"
  - "Shell-only background work (cards 16, 17): do hibernate confirmation and shuttle's waiting turn end settle it without holding a row, and do forks still get waited out?"
  - "Test economy: do new tests duplicate an existing test's coverage, and does any added behavior lack coverage?"
---

Seed focus drawn from the plan's card index and its PATTERN changes.
