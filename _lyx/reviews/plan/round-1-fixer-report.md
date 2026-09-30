# Plan-Review round 1 — fixer report

## Fixed

- **F1 (BLOCKING), card 2:**
  The no-checkout fork and the drop now live in a new unexported function in `forkrecords.go`, which `Add`'s step 8 calls in place of `createWeftWorktree`.
  `createWeftWorktree` is no longer an Edit target, and it stays as it is for `Reconcile`'s `createDormantWeftForRawWarp`.
  The card explains why: that caller never commits, and the decision record limits the drop to `Add`.
  The card also no longer asks for an update to `createWeftWorktree`'s doc comment.
  `containedWorktreeAdd` was added to Uses, and the ImpactSummary now states that Reconcile is unchanged.
- **F2 (LOW), card 2:**
  The write-out now runs `git checkout-index -a -u` in both branches and restores the root with `git read-tree -m HEAD`, and the card says why: the index keeps the stat data a full checkout would leave.
  I checked this sequence in a scratch repo before changing the card.
- **F3 (LOW), card 3:**
  I removed the `deps.go` header from the doc-update sentence, because the header stays accurate and no target covers it.

## Deferred

None.
