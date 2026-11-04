# Measure agent-turns and the batching hook from landed runs

`turn-savings-measure` · Task · research, tools, claude, shuttle · running · Worktree-Create

Measure, from transcripts of runs that already landed, the effect of project `agent-turns` as a whole and of the batching breach hook (`batching-breach-nudge`, landed 2026-11-02). One report under `docs/research/`.

## Sample (operator, 2026-11-04)

- At most 20 darn runs and 5 tapestry runs, the most recent that landed since 2026-11-02; darns weigh most, since they run most.
- No new agent run: the work reads transcripts, run records and hook state of sessions that already ran.
- The operator chose to measure the hook now rather than wait to 2026-11-16: the report states the window, and whether the hook's `batching_nudge` thresholds changed within it.
- The report states the sample size beside each number.

## Part 1: agent-turns

- Baseline: the numbers in `agent-turns`' body (`tools/tokencount` and `tools/forkaudit` on `warp-visibility`, `interactive-recipe` and `run-verbs-shared`), and `docs/research/review-turn-audit-2026-11-01` for the review roles.
- Darn writers: turns, calls per turn, shell edits against Edit-tool edits and gate-loop turns, with the scripts of `docs/research/turn-audit-2026-10-28`.
- Webster forks: turns per card, calls per turn and peak context, with `tools/forkaudit`; cost with `tools/tokencount`.
- Review roles: calls per turn, fork wake-up turns and removable turns, with the scripts of `docs/research/review-turn-audit-2026-11-01`, including the fanned Plan reviewer's pre-read turns (`plan-reviewer-batching`) against the baseline.
- Refused turns: the baseline is 6 turns refused by lym's guards across the five heaviest forks; count them again, and count `awk ... > file && mv` edits, the route the shell-edit guard leaves open.
- Card gate fix loops: before `agent-turns` one Webster fork spent 69 turns fixing its own card gate at 150k to 450k context a turn. Count, per fork, the turns between a failing card gate and its next green one, with their tokens and peak context.
- Name which members' behaviour shows in the transcripts.

## Part 2: the batching breach hook

The hook is a PostToolUse hook on Read, Grep, Glob, Edit and Write that adds one `additionalContext` line on a breach (single-lookup chain, Read slice, edit-only chain); its thresholds are `claude.yaml`'s `batching_nudge` block, copied per run into `batching-nudge.json` beside the events file with the nudge count.

- Baseline: the 28-session sample behind `batching-breach-nudge`: single-lookup turns following another single lookup above 150k context, 128 turns and 34.7M context tokens; edits spread over consecutive turns, 39 turns and 15M; of the sample's 147M.
- Measure the same two figures and every session's context tokens over the sample, extending `tools/tokencount` where it lacks the breach counts; compare by recipe and role.
- Read how often the hook fired, how often the next message after a firing batched, and how many firings were on a lookup that had to be sequential.
- Any threshold change the report recommends becomes a note of its own.
