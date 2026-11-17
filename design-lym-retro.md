# lym retro: one command that audits recent runs from their transcripts and proposes fixes

`lym-retro` · Task · feature, friction, report, tools · running · Describe

## Seen (operator, 2026-11-16)

Matt Pocock's skill `retro` (`/home/knatte/Code/pocock-skills/skills/engineering/retro/`) is one command:
it reads a session's transcript and proposes changes to the agent's environment, most severe first, and changes nothing until the user picks one.
It looks for anything: navigation, missing checks, coding standards, AGENTS.md size, tool economy, no-op instructions, information access.

lym already looks back in two ways, neither of them one command:

- Friction-Reflect, the last row of every run, files self-report issues about lym from notes the agents chose to write; it never reads a transcript.
- Transcript audits under `docs/research/` (`turns/turn-audit-2026-10-28`, `loom/darn-writer-anatomy-2026-11-08`, `turns/batching-effect-audit-2026-11-01`, `turns/writer-lookups-measure-2026-11-04`, `loom/plan-writer-audit-2026-10-24`, the glyph audits), built on `tools/tokencount` and `tools/forkaudit`.
  Each was a research task the orch set up by hand, with its own scope and scripts.
  They are scoped narrower than Pocock's: where turns, tokens and money go, and whether a stencil sentence changed behaviour.

## Design (operator, 2026-11-16)

`lym retro` files and releases a darn `docs` task; the darn does the audit in two parts.

1. The command.
   `lym retro` with no argument covers the runs landed since the last retro; it also takes slugs or a date range.
   It files one darn `docs` task from a stencil, with the runs, an exact cap on what it reads, and the question set below, and releases it.
2. The scripted part.
   The darn writer first runs the retro scripts over the runs' transcripts and records:
   `tools/tokencount`, `tools/forkaudit`, and the scripts of the past audits that become the command's own.
   They give the figures: turns and cost per producer against their medians; serial single reads and searches; commands a guard refused; verify-gate failures and fixer rounds; review rounds and recurring finding kinds.
3. The LLM part.
   The writer then reads what the figures point at, in the transcripts, stencils, heddle passages and `CLAUDE.local.md`,
   and decides what should change and where it goes, as Pocock routes it:
   a mechanical fault becomes a check (a guard, a lint, a test), a judgement call a reviewer rubric line, an instruction with no effect a deletion;
   never a new prompt sentence by default.
4. The output.
   A stamped report under `docs/research/retro/` with the figures and the proposals, most severe first, each traced to a moment in a named run.
   The operator picks which proposals become board notes; the run files none itself.

## For the run to settle (operator, 2026-11-16)

The operator leaves these to the task's own research; its Discussion reads the past audits and their scripts and decides.

- The question set and its scripts: which of the past audits' scripts become the command's own, and in which package or tool they live.
- How `lym retro` knows the last retro: the latest report's stamp, or a record.
- How Friction-Reflect relates: kept as it is, or folded in.
