# Bench Codex block v2 on P2: one high run against two medium

Adds `docs/research/codex-v2-p2-bench-2026-10-27/` with its report and `data/`.
The bench runs the forked Astra lens fan on plan `P2` with block v2 and a fork cap of 20 tool calls, once at high effort and twice at medium, and grades the findings against the Claude fan.
No code under `internal/` or `cmd/` changes.

## What the report finds

- Each run found 1 or 2 of the Claude fan's 13 findings; together the three found 3 of 13 and all 3 `BLOCKING` ones, each by a different set of runs.
- The high run and the union of the two medium runs both found 2 of 13 and 2 of 3 `BLOCKING`; the union returned 7 distinct findings against 5, at 429k fresh input tokens against 290k and 370 s against 673 s with the mediums in parallel.
- The two medium runs differ by one fan finding and share 3 of 7 distinct findings, so one run per setting does not rank the settings.
- The forks ran 2 to 7 commands each against a cap of 20, and the `way-forward` fork returned nothing in any run.
- All three runs share a finding the fan lacks: the new status and `Unit` readers of the plan's cards are missing from `allowedStatusConsumers`.

## Design decisions

- The harness is the tuned-prompt bench's, copied into `data/harness/` with its root moved to this worktree; `run-forkfan.sh` takes the run and effort, and each run has its own `CODEX_HOME` so concurrent runs' rollouts stay apart.
- `build-forkfan.sh` changes the cap sentence's number and fails when the old number remains; block v2 is in `data/prompts/block-v2.md`, equal to the board entry's.
- The three runs started together so the two medium runs count as run in parallel; the weekly window reached 54 percent at most, under the 70 percent stop, and is shared with the sibling benches.
- Grading is by one Claude session with the earlier benches' labels; the deferred-docs finding of card 14 is graded `unique-real` and marked arguable.
