# Round 3 fixer-report — plan

## Changes

- **F1** (MEDIUM, scope), fixed as proposed in `_lyx/plan/17-creel-read-sets.md`'s Intent.
  The `Read_SetsAndInbox` sequence now opens with `b1` batten, `a1` alpha, `b2` batten, `a2` alpha, `a3` alpha, with `Inbox` at batten 2 and alpha 3 before the first read.
  The first plain read gives `b1`, `b2`, the next `a1`, `a2`, `a3`, a further one nothing.
  One added sentence states why: the oldest sender holds fewer unread messages, so a busiest-sender rule fails the first read.
  The later messages are relabelled `a4`, `b3`, `a5`, `b4`, `b5`, and the `from` plus `all` step's old-set clause now names the read `b1`, `b2` and `b3`.
  The rest of the sequence's steps are unchanged.
- **F2** (LOW, consistency), fixed as proposed.
  The clause claiming the protocol row fails the old read-plus-unread set is dropped.
  The Intent now says the row fails a plain read, which hands out one sender's body only, and so fails an `All` dropped before `Store.Read`, and that the `Read_SetsAndInbox` sequence pins that no read message comes back.

## Disputed

None.

## Deferred

None.
