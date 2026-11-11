# Tune the darn writer prompt for an Opus advisor on one landed darn task

`advisor-model-trial` · Task · research, darn, loom, shuttle, claude · running · Preflight

Imported from [issue #109](https://github.com/Knatte18/loomyard/issues/109).

## Idea (operator, 2026-11-10 and 2026-11-11)

Claude Code's `/advisor` command lets a session on a cheaper model consult a stronger model when it is unsure.
The question is whether the darn writer, Darn-Write, can run on Sonnet with Opus as its advisor instead of on Opus all the way, at no loss of quality.
The operator expects darn to gain the most, since its writer does most of a darn run's work and time, and does not trust a plain Sonnet darn writer: the advisor is what could make one trustworthy.
The advisor takes effect only if the main session's prompt steers it, so the prompt is tuned before any bench.
This task is that tuning; the bench is the note `advisor-model-bench` and runs only after the operator approves the tuned prompt.

## This task: tune the writer's prompt (operator, 2026-11-11)

Iterate a prompt addition for the darn writer that says when to consult the advisor and what to ask, on one landed darn task, until the advisor is used at the right moments.

- The task rerun is `reed-tab-and-attach-fixes` (`aeea549d`, about 200 changed lines), from its base commit, in a worktree of the run's own that lands nothing.
  Each iteration is one rerun with a Sonnet writer and an Opus advisor.
- The cap is exactly five iterations, one rerun each, and no repeats of an iteration.
  The run stops earlier when two iterations in a row show no change in how the advisor is used.
- Each iteration records the prompt text it used, how often and at which moments the advisor was consulted, the review findings and bounces of the rerun against the recorded Opus run, and the tokens, cost and time with sleep removed.
- The run also confirms the advisor's settings key, that it takes effect in a lym tmux session, and how advisor calls show in the usage records.
- It passes the advisor through the `--settings` document claudeengine already writes for every launch; no lym config key is added.
- The report under `docs/research/` reads the earlier seats result first (below), gives the final prompt text and every iteration, and says whether the prompt is ready to bench.
- Runs as darn `docs`.

The risk to watch: a weak writer that is sure and wrong does not ask its advisor, so the rerun's review bounces are the key measure.

## Where an advisor may pay

Each is a role that runs on a strong model today and spends much of its tokens on routine reading and editing,
with a few hard calls an advisor could take:

- Darn-Write, the darn writer: the first place to try.
- The Code-Review fixer and the Rework writer, which apply findings already named.
- Webster's implementer forks in tapestry, which build one batch of approved cards.
- The Plan writer, from an approved decision record.
- The Discussion writer.
- Raddle's split writer.
- Describe and Friction-Reflect, which may need no advisor at all, only a cheaper model.

Review judges, lenses and the arbiter stay out: their value is the judgment itself, which an advisor would only move.

## What was tried before (operator, 2026-11-11)

lym already tried the reverse: an Opus Discussion writer as chair with one or more Sonnet assistants in seats beside it.
It gained nothing and only added cost, since the assistants came on top of a full Opus session.
The advisor differs in where the saving comes from: the cheap model does the whole session and the strong one is paid only per consult.

## Operator view (2026-11-11)

Little faith in it: an advisor needs much tuning and depends wholly on the main session's prompt.
