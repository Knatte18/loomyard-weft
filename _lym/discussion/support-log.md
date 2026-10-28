# Support log — hub-pull-verb

## Interview

A darn run holds no interview; the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- A separate verb such as `lym fabric pull-hub`: a second pull verb splits what the operator looks for under one name.
- A non-blocking parent lock attempt that reports a held lock as a stop: TryAcquire keeps a ticket when not granted, and the queue already orders waiters fairly; waiting matches `lym fabric anchor`.
- Holding the board write lock across every side: a landing never needs it to move a parent, and holding it for minutes would stall every board write in the hub.
- Pulling task pairs too: their runs keep them level, and a pull under a running row could move a branch a run is writing.

## Review rounds

_No rounds yet._

## Question ledger

- Flag or new verb? Self-picked: `--hub` on `lym fabric pull`.
- Wait for a held parent lock or stop? Self-picked: wait, as `lym fabric anchor` does.
- Beams owned by another machine? Self-picked: pulled, since a pull only reads origin.
- A beam with no upstream? Self-picked: up to date, never a stop.
