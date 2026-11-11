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

## How it runs (operator, 2026-11-11)

- The trial needs no lym config: it passes the advisor through the `--settings` document claudeengine already writes for every launch.
  Claude Code's session transcripts carry an `advisorModel` field (lym's run sessions already read `claude-opus-5-5` there), so the setting exists;
  the trial confirms its settings key, that it takes effect in a lym tmux session, and how advisor calls show in the usage records.
- A lym config key for the advisor, beside each role's model and effort, comes only after the trial shows it pays.

## Trial (orch's proposal)

- Replay rather than new tasks: rerun one or two darn tasks that already landed, from their base commit, with a Sonnet writer and an Opus advisor.
  Compare each with its recorded Opus run: review findings, bounces, tokens, cost and time with sleep removed.
- Order: the Code-Review fixer first (low risk, it applies findings already named), then Darn-Write.
- The risk to watch: a weak writer that is sure and wrong does not ask its advisor, so the review bounces are the key measure.
- Runs as darn `docs`; the task body states the exact number of runs.
