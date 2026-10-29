# Support log — chases-refused-forward-journal

## Interview

A darn run holds no interview.

## Rejected alternatives

- Hold alpha by `program(batten)`, as the inject-TTL row does: rejected, since a hold by anyone but the message's sender stops the chase clock, so no chase is ever posted.
- Keep beta as sender and let beta hold alpha: rejected, since `Hold` refuses a non-privileged strand holding another strand.
- Wait for the worker with `awaitChaseLines` and refuse the forward of an already-read chase: rejected, since it stops covering the unread case the row names; the entry names it as the fallback only.

## Review rounds

_No rounds yet._

## Question ledger

- Does the chase clock reach its first chase under a hold from another sender? Resolved from the code: no, a hold stops the clock of every message not from its owner; self-picked the question's sender as the hold owner (D1).
- Does the row still need `awaitChaseLines`? Self-picked no (D2).
