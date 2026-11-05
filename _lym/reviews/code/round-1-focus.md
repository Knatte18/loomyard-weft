---
round: 1
focus:
  - "internal/fabricengine/remove.go, destroy.go: does `lym fabric remove` delete the whole task worktree directory, ignored build output included, and return an error naming the directory when the delete fails? Check the order: teardown finishes first, then the stray delete."
  - "internal/fabricengine/destroy.go `RemoveStrayTaskPath`: can it delete a directory that is a real worktree, a path outside the task worktree, or one a different slug owns? Settle whether the 'caller knows the pair stood there' precondition is enforced or only assumed."
  - "internal/pairteardown/teardown.go `ClearStray`: does it run only after the pair is verified gone, and does a failed delete fail the teardown on every path, including a resumed teardown?"
  - "internal/battencli/cancel.go, wire.go, beamland.go: does `lym batten cancel --remove` on a slug whose worktree is gone but whose directory remains succeed instead of erroring 'not a git repository'? Check the cancel's left-count and the beam-land change that reads a pair only from a registered worktree."
  - "Goal part 2: does the board's 'has a pair' check now treat a non-worktree directory as no pair? Find the check and settle whether the diff changes it or leaves it unchanged."
  - "internal/fabricengine/pairparts.go: does the new `StrayPath` report change what other `PairParts` consumers (status, cleanup, pairs listing) treat as a pair?"
  - "Test coverage: the new tests in remove_leftoverdir_integration_test.go, teardown_test.go, cancel_test.go and wire_test.go against each new behavior; check for duplicate coverage and for behavior the diff adds with no test."
  - "pattern/PATTERN-pair-teardown.md and the doc.go changes: the repository's own rules for docs and pattern entries that a changed teardown contract requires."
---

Range is `b261d82c..HEAD`, three commits. The answer key is the decision record's Goal alone, with no plan.
The Goal's second fault names two expectations: the board's has-a-pair check, and `cancel --remove`.
The diff touches no board package, so the first expectation is the likeliest scope gap to settle.
