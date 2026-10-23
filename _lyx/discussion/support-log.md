# Support log — orch handoff timeout counts from delivery

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Keep a backstop timeout from the post, such as twice the handoff timeout, while the request is unread: it brings back the reported failure for a turn longer than the backstop, and an unread request blocks nothing while the orch is busy.
- Count from creel's first typing of a notice line instead of the read: the note request is mail, whose delivery the journal records as its `read` event, and the board entry names that event.
- Keep the requested cycle's request-time start: a requested cycle queued behind a turn fails the same way, and the idle phase's stale-marker removal still bounds a request that never starts.
- Query the journal on every tick without persisting the result: a restarted watcher would still find it, but the query repeats for the whole phase and the state file would not show the read.

## Review rounds

_No rounds yet._

## Question ledger

- Should the clock start at the read for every trigger, or only for soft and hard ones? Self-picked: every trigger (D1).
- Should an unread request ever time out? Self-picked: no (D2), bounded by the strand check and the hold's effect.
- Where is the read time kept? Self-picked: a new `State.RequestReadAt`, cleared with `State.RequestID` (D3).
