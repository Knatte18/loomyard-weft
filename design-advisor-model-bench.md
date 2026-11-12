# Bench a Sonnet darn writer with an Opus advisor on three landed darn tasks

`advisor-model-bench` · Task · research, darn, loom, shuttle, claude · draft

Depends on: [`shuttle-run-advisor-option`](design-shuttle-run-advisor-option.md)

## Idea (operator, 2026-11-11)

The bench that follows `advisor-model-trial`, which tunes the darn writer's advisor prompt.
It is filed as a task only when the operator approves that tuned prompt.

## State after the trial (board tidy, 2026-11-11)

`advisor-model-trial` landed as a report only (`3671f3d4`, `docs/research/loom/advisor-model-trial-2026-11-11/report.md`).
It ran no iteration and made no tuned prompt, so there is no prompt to approve and this bench is not ready to run.
The blocker: no route sets Claude Code's advisor (`advisorModel` setting, `--advisor` flag) for one shuttle run.
The operator decides the way forward: a per-run advisor option on `lym shuttle run`, or another route.
The bench waits on that and on a tuned prompt.

## What it runs

Rerun three landed darn tasks of different sizes from their base commits, with a Sonnet writer, an Opus advisor and the tuned prompt, identical in all three:

- large: `decision-record-replace-verb` (`6d58cad7`, about 2970 changed lines);
- middle: `adopt-takeover-fixes` (`6f8d4cff`, about 580 changed lines);
- small: `reed-tab-and-attach-fixes` (`aeea549d`, about 200 changed lines).

The cap is exactly three runs, one per task, and no repeats.
Each run is compared with the task's recorded Opus run, whose recipe and profile come from its usage record:
review findings, bounces, tokens, cost and time with sleep removed.
It runs as darn `docs`, and the report goes under `docs/research/`.

## Not decided

Whether a lym config key for the advisor, beside each role's model and effort, follows: only after the bench shows it pays.
