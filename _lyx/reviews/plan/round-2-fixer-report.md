# Round 2 fixer-report — plan

## Changes

- **F1** (BLOCKING), fixed as proposed in `_lyx/plan/17-creel-read-sets.md`.
  The premise holds: `resolveSender` (`internal/creelengine/identity.go`) refuses a program claim from a pane that is not an orch strand's, and with `asStrand` true `newRig` makes the test process alpha's pane process.
  The `ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread` row now stays on `asStrand` true and only loses its `read --all` assertion.
  A new row, `ReadAllHandsOutEveryUnreadMessageFromEverySender`, runs with `asStrand` false.
  It makes `ly:BATTEN`'s mailbox known, has an external client and a `ly:CARTER` program client each post to it, and asserts that a `Request` with `All` set hands out exactly the two bodies, in post order.
  The Intent says why the row is `asStrand` false and why it asserts exactly two bodies.
- **F2** (LOW), fixed as proposed.
  The `Read_SetsAndInbox` description is now an explicit sequence of posts and reads with distinct bodies (`a1`–`a4`, `b1`–`b4`), and each read has an exact want.
  The `from` step names the sender that is not the oldest.
  The `from`-plus-`all` step runs while read batten messages are in the pool and alpha messages are unread, so it fails both the old set and a missing narrowing.
- **F3** (LOW), fixed as proposed.
  Added `internal/creelengine#Reading` to `Edit:`.
  The Intent now names the rewording of `Reading`'s doc comment, of `Store.Read`'s second comment line and of `doc.go`'s second read-paragraph sentence.
  The closing grep also searches for `unread ones` over `internal/creelengine` and lists `verbs.go`'s `Reading` comment.
  Running that grep against the current tree hits exactly `doc.go:76`, `verbs.go:121` and `verbs.go:131`.
  I did not rerun the overlay check that the review cites.

## Disputed

None.

## Deferred

None.
