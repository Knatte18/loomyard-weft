---
round: 2
exclude_lenses: []
focus:
  - "Decision record D3, D5, D7, D8: wherever a background pusher pushes a run's records, check which failed-push records it can write, whether those records make the act rule hold the run, and whether a pause covering the run stops the pusher on every path that starts or continues it."
  - "Decision record D3 and D6: wherever the prime's records pusher serves several batten runs, check how it learns of another machine's takeover of each run before it replays commits, and which fabric site reads it."
  - "Decision record D3, D5, D6: wherever a pusher is registered, restarted or handed over, check its runproc owner key against the slug-keyed teardown and cancel paths, and the hand-over's pair-removal flag."
---

Round 1's findings all fell in one or more lenses at MEDIUM or worse, so no lens is excluded.
