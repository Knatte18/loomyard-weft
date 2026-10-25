# Bench Codex Astra against solo Opus as a plan reviewer

Adds `docs/research/codex-astra-solo-bench-2026-10-25/`: a report and its data for four reviews, `gpt-6-astra` and Claude `opus` at medium effort, each solo with no lens forks, on the plans `P1` (`gate-runs-once`) and `P4` (`machine-handover`) of the earlier Codex bench.
It is a documentation-only change; no code under `internal/`, `cmd/` or `contracts/` changes.

## Result

On these two plans Astra and solo Opus each reproduced 4 of the Claude fan's 23 findings.
Opus reproduced 1 of the fan's 5 `BLOCKING` findings, Astra none.
7 of Opus's 12 findings and 2 of Astra's 6 are real on reading and absent from the fan's list, and none of either was graded false.
Astra was 2.3 to 3 times faster (166 s and 184 s against 389 s and 563 s) and cost 22 points of the Plus plan's 5-hour window on `P1` and at most 15 on `P4`.
Opus approved `P4`, where the fan blocked; Astra blocked both plans as the fan did.
The sample is two inputs with one run per cell and one grader, so it ranks neither reviewer.

## What the change adds

- `report.md`: conclusion, summary table, per-input findings set against the recorded fan review, method, deviations and limits.
- `data/match.tsv`: the grading of every finding.
- `data/usage.txt`, `data/inputs.txt`: tokens, wall time, window readings and the input commits.
- `data/reviews/`: the four review files.
- `data/harness/`: the earlier bench's `build.sh`, `run.sh`, `setup-codex-home.sh`, `windows.sh` and `dump.sh` retargeted at this worktree, and `run-opus.sh`, which runs the Opus review through `lyx shuttle run`.

## Design decisions

- Opus runs as an interactive shuttle session, not `claude -p`, per the repository's rule that agents run in interactive tmux.
  The session loads this worktree's `CLAUDE.md`, which the Codex run does not; the report lists it as a limit.
- The 5-hour window could not be read before the first Codex run, since only a rollout holds a reading and the newest one was over a day old.
  The first run served as the reading; the report states this as a deviation.
  No run was refused or cut off, and the 70 percent stop was not reached.
- Opus token figures come from the session transcript, once per assistant message id, and count cache creation as fresh input; the report says the figure is not the same measure as Codex's.
- The Opus run on `P1` was started in the background by the harness and finished normally; no run was repeated.

## Verification

The change touches only `docs/research/` and gitignored `.scratch/`, so no test run applies.
`lyx loom validate-discussion` and `lyx loom validate-description` pass.
