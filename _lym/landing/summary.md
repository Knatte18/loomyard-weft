# Audit where agent turns go and which could be merged

Adds `docs/research/turn-audit-2026-10-28/`: a report on turns per session, mergeable serial turns and the read-edit-test pattern per role, with the scripts and TSV tables that produced it.
It changes no code, stencil, system prompt, profile or config; the recommendations in the report are not decisions.

## What the report finds

- The sample is the 213 sessions (93 main, 120 forks) on this machine whose runs landed since 2026-10-20: one loom run and nine darn runs.
  The other landed runs' transcripts are not here, so every loom-only role rests on one to three sessions.
- 72% of turns carry one tool call.
  The darn writer is at 86% (n = 14), the Plan writer's lens forks at 90% (n = 6).
- A strict estimate finds 582 removable serial gather turns, 22% of main-session weight and 16% of fork weight.
  The darn writer carries 11.9 per session (20% of its weight, n = 14), the Plan reviewer 17.7 (46%, n = 3), the Discussion writer 23.0 (42%, n = 1).
- Serial calls are shell `grep` (46%) and `sed -n` (26%); the Read tool is 2% of them.
- Reads continue after the first edit (12.5 per darn session, 33.8 per Webster fork), and edits run near one per turn.
- The report words one stencil sentence per role, and one system-prompt sentence for all roles, each with a ceiling and a halved estimate of turns and weight saved.

## Design decisions

- A turn is one assistant message id; cost is the `tools/tokencount` weight, since the records hold no price table.
- Mergeable calls are read-only gathers; gate, test, lint and validate calls are never counted, since a repeat follows an edit or a busy gate slot.
- The strict count treats a call as dependent when it names a path-like token that first appears in the previous turn's results; the class-only count is kept beside it as the upper bound.
- The scripts use `bash`, `jq` and `awk`, since `go run` is refused in this session; `tools/tokencount` was not run and its calibration is not reused.

## Checks

`lym gate test --wait` ran green on the docs-only diff.
The scripts were run once end to end through `run-all.sh`; the committed TSV files are that run.
