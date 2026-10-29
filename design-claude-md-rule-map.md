# Map every rule of the loomyard CLAUDE.md to where lym agents already get it

`claude-md-rule-map` · Task · research, pattern, stencil · draft

## Goal

Map every rule of the loomyard `CLAUDE.md` to where lym's agents already get it, so the operator can decide per rule what happens to it before the file is deleted (`claude-md-lym-rules-out`).

## Scope

- Input: the repository's `CLAUDE.md`, one file, read by one writer session in one run; no other runs, models or repeats.
- The writer reads the source itself: `CLAUDE.md`, `PATTERN.md`, the `pattern/` files, `contracts/stencils/`, `plugins/` and the orch rules.
- It first records which roles receive `PATTERN.md` by injection and which do not, citing the code that injects it.
- Then one table row per rule or sentence of `CLAUDE.md` with the columns: the rule in a short quote, where it already stands for the agents that need it (`file:line`, or `none`), the agents that need it, and a verdict of `covered`, `move to <place>` or `drop`, each with a one-line reason.
- The report lands as `docs/research/claude-md-rule-map-2026-10-29.md` and changes no other file; `CLAUDE.md` stays as it is.
- It names every rule whose only home is `CLAUDE.md` and which agents would lose it on deletion, and every rule enforced in code, with the check that enforces it.

## What the run proves

The operator's choice per row is taken from the table, and the follow-up task moves, drops or guards each rule as decided and deletes the file.
