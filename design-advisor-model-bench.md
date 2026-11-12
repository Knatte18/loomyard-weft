# Bench a Sonnet darn writer with an Opus advisor on three landed darn tasks

`advisor-model-bench` · Task · research, darn, loom, shuttle, claude · running · Darn-Write

## Idea (operator, 2026-11-11)

Whether a Sonnet darn writer with an Opus advisor matches an Opus writer on a landed darn task, at lower cost.
`advisor-model-trial` stopped for want of a per-run advisor option on `lym shuttle run`; the task `shuttle-run-advisor-option` adds it, and this bench runs after it.

## What it runs

1. The advisor prompt: before any run, the task writes the prompt addition that tells the Sonnet writer when to consult the advisor and what to ask.
   It states the prompt in full at the top of the report, and uses it unchanged for all three runs.
   `docs/research/loom/advisor-model-trial-2026-11-11/report.md` holds the setup that trial reached (the recorded run, the base-commit checkout and the stand-in review).
2. Rerun three landed darn tasks of different sizes from their base commits, with a Sonnet writer and an Opus advisor set through the per-run option:
   - large: `decision-record-replace-verb` (`6d58cad7`, about 2970 changed lines);
   - middle: `adopt-takeover-fixes` (`6f8d4cff`, about 580 changed lines);
   - small: `reed-tab-and-attach-fixes` (`aeea549d`, about 200 changed lines).
3. Compare each run with the task's recorded Opus run, whose recipe and profile come from its usage record:
   review findings, bounces, tokens, cost and time with sleep removed, and how many advisor consults the writer made and what they cost.

## Cap

Exactly three runs, one per task, and no repeats; no prompt iteration.
The report goes under `docs/research/`.

## Not decided

Whether a lym config key for the advisor, beside each role's model and effort, follows: only after the bench shows it pays.
