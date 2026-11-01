# Fanned reviewer gathers its fork pre-read in waves of parallel calls

The round-1 Plan reviewer on Opus made 1.05 calls a turn over three runs, and 41% of its weight sat in turns a merge would remove (`docs/research/review-turn-audit-2026-11-01/report.md`).
It already carried the batching sentence of `bf00a043`.

## Cause

Splitting each reviewer session of the audit's `calls.tsv` at its first `Agent` call puts the serial turns before the fork spawn:

| Role | Turns before the spawn | Calls a turn before | Calls a turn after |
| --- | --- | --- | --- |
| Plan reviewer, round 1, Opus (3 sessions) | 31, 60, 64 | 1.03 to 1.10 | 1.17 to 1.24 |
| Plan reviewer, later rounds, Sonnet (4 sessions, no spawn) | 52 to 85 | 1.29 to 1.64 | – |
| Discussion reviewer, Opus (8 sessions) | 8 to 38 | 1.08 to 1.60 | 1.46 to 2.29 |

The turns before the spawn are the pre-read that `burler-fork-reading` asks of a fanned round's handler.
That stencil is a checklist walked per symbol: callers, callees, writers, tests and restart paths of each thing the target changes.
It carries no batching line.
The explore step's batching sentence covers only the target's own reads.
A Plan names many more symbols than a decision record, so the walk runs longest in the Plan round.
The Sonnet rounds are solo, so they never get the block.

## What changes

- `contracts/stencils/burler/burler-fork-reading.md` now says how to gather the checklist, in waves of parallel calls:
  first one message with a call for each file the target names, one search per symbol and per state, `lym board get` and each `pattern/` background file;
  then one message for the definitions and call sites those results point to;
  then a further wave only for a new hop, such as a spawned process or restart path.
  A call goes to a later message only when its arguments come from an earlier result.
- `internal/burlerengine/doc.go` describes the waves.

The checklist, the no-judgement rule and the reading manifest are unchanged.
The block renders only in a fanned round, so solo rounds see the same prompt as before.

## Design decisions

- **One trial: the instruction, not the model.**
  The entry asks for one change at a time.
  The instruction goes first because the data puts the serial turns in a block the Sonnet rounds never see.
  Comparing round 1 to the later rounds therefore mixes the model with the block, and running round 1 on Sonnet would change both at once.
  If the instruction leaves the pre-read serial, the model swap is the next trial.
- **A stencil fix, not a heddle line.**
  The cause is in lym's own reviewer instructions, which belong in its stencils (`PATTERN-lym-instructions-in-code`).
- **Measured after landing.**
  The measure is the Plan reviewer's calls per turn and removable turns.
  To measure, add the slugs of the next fanned Plan rounds to `docs/research/review-turn-audit-2026-11-01/slugs.txt` and rerun its `run-all.sh`.
  The figures to beat are the turns before the spawn in the table above, and the reviewer's 33.3 removable turns a session.
  No measurement is part of this change, since it needs new runs.
- The Bolt checkout's seeded copy of the stencil changes on the next `lym stencil sync`.
