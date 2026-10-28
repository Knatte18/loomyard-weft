# Support log — spawn-text-file-busy

## Interview

A darn run holds no interview; the writer read the board entry and the code and made the calls in the decision record.

## Rejected alternatives

- Retrying in `proc.StartTied` for every caller: other callers start other programs, and the loop is the site the issue names; the change stays where the failure was seen.
- A new transient class for ETXTBSY so the loop re-steps once: the start retry already absorbs the window, and a re-step after the retry bound would only repeat it.
- A new anomaly kind for a failed start: the cause and trace file make the note self-explanatory without a new kind for the reflection to learn.
- Installing through `lymtemp`'s run dir: it can sit on another filesystem than the Go bin dir, where a rename is a copy.

## Review rounds

_No rounds yet._

## Question ledger

- Which spawn sites retry? Self-pick: the loop child start only, where the issue's failure occurred.
- Which deploy paths change? Self-pick: `tools/deploy`, which every named path runs.
- What retry bound? Self-pick: five tries, 100 ms first wait, doubling.
