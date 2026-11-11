---
round: 2
exclude_lenses: []
focus:
  - "D8's off-switch migration for the retired key test_map_every: check whether the mechanism runs before configsync.ReconcileHubWideAt prunes retired keys (internal/hubreconcile/walk.go order against internal/configsync/configsync.go pruneSparse), and whether the prime worktree's and local override layers of the per-worktree beam module are covered or bounded."
  - "D1's verdict list against D4, D8 and D10: settle whether every declared mapped test with no entry is queued again after an ended pass, a discarded round or a dropped outcome entry, so the backlog drains without a next Affected verdict."
  - "D8 and D9 on a hub with fewer than two slots, and the yield-versus-hand-call lock wait: settle whether any pass restarts at the same HEAD without progress and whether a yielded pass and a hand call can wait on each other; also settle how the verb tells a dobby-started pass from a hand call and what bounds that marker."
  - "D11 and D10 against the code: settle which component ends a test-map pass at an anchor move (internal/fabriccli anchor wait and EndDobbies) and whether the side-by-side parallel rule selects any test given that every test opens with tiergate.Require."
---

Round 2 re-examines the areas where round 1 found design flaws; the record's text there is new.
