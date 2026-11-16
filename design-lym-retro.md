# lym retro: one command that audits recent runs from their transcripts and proposes fixes

`lym-retro` · Note · feature, friction, report, tools

## Seen (operator, 2026-11-16)

Matt Pocock's skill `retro` (`/home/knatte/Code/pocock-skills/skills/engineering/retro/`) is one command:
it reads a session's transcript and proposes changes to the agent's environment, most severe first, and changes nothing until the user picks one.
It looks for anything: navigation, missing checks, coding standards, AGENTS.md size, tool economy, no-op instructions, information access.

lym already looks back in two ways, neither of them one command:

- Friction-Reflect, the last row of every run, files self-report issues about lym from notes the agents chose to write; it never reads a transcript.
- Transcript audits under `docs/research/` (`turns/turn-audit-2026-10-28`, `loom/darn-writer-anatomy-2026-11-08`, `turns/batching-effect-audit-2026-11-01`, `turns/writer-lookups-measure-2026-11-04`, `loom/plan-writer-audit-2026-10-24`, the glyph audits), built on `tools/tokencount` and `tools/forkaudit`.
  Each was a research task the orch set up by hand, with its own scope and scripts.
  They are scoped narrower than Pocock's: where turns, tokens and money go, and whether a stencil sentence changed behaviour.

## Idea

A `lym retro` command that runs such an audit without a hand-written task:

- Input: the runs landed since the last retro by default, or named slugs or a date range.
- It reads the runs' transcripts and records with `tokencount` and `forkaudit` and a fixed set of lym-scoped questions, such as:
  turns and cost per producer against their medians; serial single reads and searches; commands a guard refused; verify-gate failures and fixer rounds; review rounds and recurring finding kinds; instructions in stencils, heddle passages and `CLAUDE.local.md` that the transcripts show had no effect (no-ops).
- It routes each finding as Pocock does: a mechanical fault becomes a check (a guard, a lint, a test), a judgement call a reviewer rubric line, a no-op a deletion; never a new sentence in a prompt by default.
- It writes a report under `docs/research/retro/` with a stamp, and proposes board notes, most severe first; the operator picks which to file.
- It runs as a darn docs task the command files and releases itself, so it lands its report like any audit, with an exact cap on what it reads.

## Open

- Whether the command files the task (and the operator still approves the notes), or runs one agent in place.
- The fixed question set and its scripts: which of the past audits' scripts become the command's own.
- How Friction-Reflect relates: kept as it is, or folded in.
