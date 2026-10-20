---
round: 4
focus:
  - "Bolt entry merge in the decision record's Bolt replay decision: the reconcile holds Bolt's `board.lock` and `.gitrepo-push.lock`, and the board's write path (`boardCriticalSection` in `internal/boardengine/board.go`) takes `board.lock` with a blocking acquire. Settle whether the record names an entry point that runs the board's validation without taking either lock, and whether it states the unit of refusal (one entry or design-file change) and what happens to a later change that depends on a refused one."
  - "Ended pair with a busy driver: the cleanup teardown runs `pairteardown`'s `EndSession`, whose refusal probe (`RemoveRefusal`, no force from cleanup) runs before the driver is ended. Settle whether the record decides what happens to a dirty task worktree for a pair held by a wake-written pair pause entry, and whether the busy notice's way forward actually succeeds."
  - "Waits the reconcile's teardown still makes: `fabricengine`'s `Remove` takes the weft write lock with a blocking acquire and waits up to 30 s on the pair's teardown lock, and `EndSession` waits up to two minutes on the loop and run locks. Settle whether the record's stated bounds list each wait and set its value or replace it with a try-acquire."
  - "Evidence that an entry was removed from origin's Bolt: settle whether the rule still works after local Bolt was fast-forwarded by an earlier pull-first, and whether every reuse of the rule states the same evidence."
  - "Rewound against rewritten branch outcomes: settle whether the two conditions are disjoint or the record states which wins, and whether each has its own Acceptance criterion."
  - "Wake under a hub pause the daemon's hibernate request does not own: settle whether the record states what the wake reconcile does at the network gate."
---

Entries restate each site they rely on, so no prior-round material is needed.
