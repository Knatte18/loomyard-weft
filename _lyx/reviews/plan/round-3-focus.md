---
round: 3
focus:
  - "Card 4 (`_lyx/plan/04-board-owner-writes.md`), `TestSetOwners` rows: for each refusal or reporting branch the card prescribes in the owner setter (a push deferred by a held lock then retried once after `Sleep`, a branch still ahead of its upstream refusing naming `lyx board sync`, a competing write replayed so origin's owner stands and the slug lands in `Lost`), settle whether the row names an input that reaches that branch. `SetOwners` pushes through `Bolt.PushRecorded` and `pushClassified` in `internal/fabricengine/pushclassified.go`: its first push is lock-free, `retryBoltPush` reports a deferral only after a rejected first push when the board write lock is held, and `PullThenCommitWritten` takes that lock and pulls immediately before the commit. A row that only holds a lock around the call, or only calls the error constructor, does not reach the branch."
  - "Card 4, any new test seam the card adds to `OwnerRequest` (a hook, a stub, a `Sleep`): settle whether the card states what it is in production (nil or a real sleep), that it is the test's only route to the branch, and what bounds it, and whether `Sleep` still follows `PATTERN-test-speed`."
  - "Cards 9 and 24 (`Lost` refusals): settle whether the path they rest on, `SetOwners` reporting a lost slug with origin's owner after a post-push re-read, is exercised by some test in card 4 and not only against a fake `Set`."
---

Round 2's single finding concerned card 4's test rows; the focus checks that the repaired rows reach their branches and that no new unreachable refusal appears.
