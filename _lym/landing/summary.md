# Survey Matt Pocock's skills repository for ideas LoomYard could use

Adds `docs/research/survey/pocock-skills-2026-11-19/report.md` and indexes it under a new `survey` section of `docs/research/README.md`.
No code, stencil or spec changes.

The report covers `mattpocock/skills` at `main` `5863806` and the unmerged `origin/constrain-skill` at `31964fa`:
every skill under `skills/` except `deprecated`, the `docs/` pages and the `constrain` skill.
Each candidate is compared with lym's skills under `plugins/`, the stencils, the recipes and `pattern/`.
The method section records the operator's lens, sent by the run's parent during the run.
A candidate scores by whether it can become structure (a verb, a row, a gate, a guard or a stencil rule), not by working as a live-chat skill.
The `git log` of each candidate gives its age, and a candidate younger than about two weeks, or one that replaced an earlier take, is flagged unproven.

The candidate table gives verdicts:

- **Adopt:** merge danger in the change description, a red-first gate with a `Root-Cause:` trailer for bug runs, and tautological tests as a Code-Review finding.
- **Adapt:** expand–contract ordering for hand-made wide changes in plans, and the ADR three-part test for `## PATTERN changes`.
  An out-of-scope match against `abandoned` board entries and a "prompt the positive" stencil rule are adapt too, at low priority or after a measurement.
- **Skip:** the Fowler smell baseline, which overlaps `scribe:code-quality`.
  Also skipped are `constrain`, `software-factory` and `chief-of-staff`: each is flagged unproven, and lym already covers its core with `scankit` allowlist tests, gates, capped Bouncer decisions and `lym retro`.

A one-line list names what LoomYard already does, such as the retro routing, Discussion's batched questions with recommended answers, and the shell guard.
The report ends with the three adopt candidates, each written so it can become a board note.

## Design decisions

- The report sits in a new `survey` topic in the research index, between `speed` and `tokens`, since no existing topic holds surveys of outside projects.
- The `docs/` pages were read for their headings and their notes on renamed, dropped or replaced skills, not in full.
  Those notes are what the lens's "has it held?" test needs, and the skills themselves were read whole.
- No board note was filed. The report writes the three candidates so the operator can file them.
