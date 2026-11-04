# Measure agent-turns and the batching hook from landed runs

Adds one research report, `docs/research/turn-savings-measure-2026-11-04/report.md`, with the scripts and tables that rebuild it from the Claude Code transcripts of runs that already landed.
It measures project `agent-turns` as a whole and the batching breach hook (`batching-breach-nudge`) over the 20 most recent darn runs and the 4 tapestry runs that landed since 2026-11-02, against the runs `agent-turns` was first measured on.
No agent run was made, and no production code changed.

## What the report finds

- `agent-turns` shows in the transcripts: Webster forks take 13.8 turns per card against 31, shell edits fall from 339 calls to 21, the darn writer issues 2.05 calls a turn against 1.08, and the card gate fix loops fall from 5 loops and 53 turns to 1 loop and 3 turns.
- The five heaviest Webster forks hold 0 guard refusals against the baseline's 6, which the same count reproduces on the baseline.
- The hook fired 4 times in 24 runs, in main sessions only.
  Its lookup chain matches the Read, Grep and Glob tools, and these sessions search with Bash `grep`, so almost no lookup chain is visible to it; forks, which hold over half of the edit-chain turns, are outside it.
- The thresholds of `batching_nudge` did not change in the window, and the report recommends no change: 4 firings are too few to tune on.
  Its two coverage questions (Bash lookups, forks) are raised for the operator and filed as no note, since neither is a threshold.

## Design decisions

- The breach counts live in new scripts in the report's folder, `breaches.sh` and `events.sh`, not in `tools/tokencount`.
  The counts read each assistant message's tool calls, the hook's attachments and the guards' refusals, none of which `tokencount` reads, and it already reads whole sessions through the shuttle engines; the turn audit's jq extractor already holds the turn and call tables the counts need.
  The decision record allowed extending `tokencount` where it lacks the counts; this is the one deviation from that wording.
- The sample is 4 tapestry runs, not 5: only four landed in the window.
  `cold-gate-fixer`, one of them, ran no Webster fork and may predate the binary that carries the hook; it stays in the sample, and the report says so.
- The baseline's figures are recomputed with the same scripts on the three runs `agent-turns` named, so a recipe-for-recipe comparison uses one definition; the hook's own quoted baseline (28 sessions, not listed) is quoted, not recomputed.
- A backgrounded card gate takes its verdict from the task notification that reports it.
  Seven card gates have none, so the report states the loop count as a lower bound, and `gate-loops.sh` has an upper-bound mode.
- No board note was filed for a threshold change, since the report recommends none.

## Verification

`run-all.sh` runs end to end on this machine and writes every table; the five-heaviest-fork refusal count reproduces the board entry's 6 on the baseline runs.
No Go code changed, so no gate applies to the diff.
