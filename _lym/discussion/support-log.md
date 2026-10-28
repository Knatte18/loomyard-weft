## Interview

A darn run holds no interview.

## Rejected alternatives

- Running `tools/tokencount`: it reads cost and time per segment, not per-turn tool calls, and `go run` is refused in this session.
- Judging mergeability by tool class alone: it overcounts reads that follow a listing; kept as an upper bound only.
- Pricing by model rates: no price table is in the records; the tokencount weight is used.

## Review rounds

_No rounds yet._

## Question ledger

- Which sessions count? Self-pick: transcripts under slugs archived since 2026-10-20, started on or after that date.
- Are bench and spike darn runs in the sample? Self-pick: yes, they are darn runs by their seed; the report labels the roles, not the tasks.
