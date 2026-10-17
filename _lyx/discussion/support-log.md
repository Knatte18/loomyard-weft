# Support log — driver-watched-verify-notice

## Interview

A darn run holds no interview; the decisions come from the board entry and a reading of `internal/sheddriver`.

## Rejected alternatives

- Reword the line to say batten retries the halt and no action is needed: every driver notice ends in the tail asking the orch to answer, so the line would still wake the orch for nothing, and a tail-less special case adds a notice form for no gain.

## Review rounds

_No rounds yet._

## Question ledger

- Drop the message or reword it? Self-picked: drop it, for the reason under `## Rejected alternatives`.
