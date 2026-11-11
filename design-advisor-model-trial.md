# Trial a Sonnet role with an Opus advisor, darn writer first, against Opus all the way

`advisor-model-trial` · Note · research, darn, loom, shuttle, claude

Imported from [issue #109](https://github.com/Knatte18/loomyard/issues/109).

## Idea (operator, 2026-11-10 and 2026-11-11)

Claude Code's `/advisor` command lets a session on a cheaper model consult a stronger model when it is unsure.
Trial whether a role can run on Sonnet with Opus as its advisor, instead of on Opus all the way, at no loss of quality.
The operator expects darn to gain the most, since its writer does most of a darn run's work and time.
The operator does not trust a plain Sonnet darn writer; the advisor is the thing that could make one trustworthy.

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
The trial's report reads the seats result first, under `docs/research/`, and says why this case may differ.

## How it runs (operator, 2026-11-11)

- The trial needs no lym config: it passes the advisor through the `--settings` document claudeengine already writes for every launch.
  Claude Code's session transcripts carry an `advisorModel` field (lym's run sessions already read `claude-opus-5-5` there), so the setting exists;
  the trial confirms its settings key, that it takes effect in a lym tmux session, and how advisor calls show in the usage records.
- A lym config key for the advisor, beside each role's model and effort, comes only after the trial shows it pays.

## Stage 1: tune the writer's prompt before any bench (operator, 2026-11-11)

The advisor takes effect only if the main session's prompt steers it: when to consult, and what to ask.
So the prompt is tuned over several iterations before the bench, and the bench runs only once the operator approves the tuned prompt.

- The tuning is an iterating run (a sampler), on the small task below, so each iteration is cheap.
- Each iteration is judged on whether the advisor is consulted at the right moments and whether the result still lands as the recorded Opus run did.
- The task body states the exact number of iterations as a cap, and where the prompt text is kept (in the report under `docs/research/`).
- This stage also confirms the settings key, that it takes effect in a lym tmux session, and how advisor calls show in the usage records.

## Stage 2: the bench (operator, 2026-11-11)

It runs after stage 1 and is its own task, filed only when the operator approves the tuned prompt.

- The trial runs against work that has already run: rerun one or two darn tasks that already landed, from their base commit, with a Sonnet writer and an Opus advisor.
  Compare each with its recorded Opus run: review findings, bounces, tokens, cost and time with sleep removed.
- The test is the darn writer, Darn-Write, since a darn is the most realistic place to test an advisor; no fixer-first stage.
- The tasks differ in size: three landed darn tasks, one run each, so three runs in all:
  - large: `decision-record-replace-verb` (`6d58cad7`, about 2970 changed lines);
  - middle: `adopt-takeover-fixes` (`6f8d4cff`, about 580 changed lines);
  - small: `reed-tab-and-attach-fixes` (`aeea549d`, about 200 changed lines).
  The run states each task's original recipe and profile from its usage record and compares against that.
- The risk to watch: a weak writer that is sure and wrong does not ask its advisor, so the review bounces are the key measure.
- Runs as darn `docs`; the cap is exactly three runs, one per task above, and no repeats, with the tuned prompt of stage 1 identical in all three.

## Operator view (2026-11-11)

Little faith in it: an advisor needs much tuning and depends wholly on the main session's prompt.
It is not planned to run until the operator releases it.
