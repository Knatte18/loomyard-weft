# Codex support in levels: review, darn, then the rest of tapestry

`codex-support-levels` · Note · research, codex, shuttle

## Goal (operator, 2026-11-09)

Lym can say it supports Codex only when Codex can take over what Claude does, not only as an assistant reviewer.
That is a large job, so "support" is defined in levels, and lym claims the highest level that is proven on real runs.

## Decision (operator, 2026-11-09)

This note takes the operator's wish for full support beside the decision of 2026-11-06 (note `codex-assist-review-trial`), which kept Codex to review and support and dropped a writer comparison because the operator doubted Codex writes in the same tone as Claude.
Level 2 below is where that doubt is tested, and the bench is its first work.

## Levels

1. **Review.** Codex as the non-gating assistant reviewer beside the Claude reviewer.
   Partly proven: runs and benches under `docs/research/codex/review/`; the open work is in `codex-assist-review-trial`.
2. **Darn.** Codex as the writer of a whole darn run that lands with no resumption.
   Open: the 2026-11-05 trial (`docs/research/codex/engine-trial/codex-engine-trial-darn-e2e-2026-11-05.md`) stopped at Darn-Write because the sandbox made the worktree's git directory read-only, so the commit was refused.
   `sandbox_mode: danger-full-access` (`codex.yaml`, set with `lym config codex --layer machine`) is built but checked by unit tests only; no real Codex session has run with it.
3. **The rest of the tapestry recipe.** Codex in Discussion, Plan, Plan-review and Code-Review as the main seat, and in raddle.
   Not tried; it waits until level 2 shows the quality holds.

## First work: the level 2 bench

A bench of Codex as the darn writer against Claude on one or two real darn tasks, each run once per writer.
It reads, per run: cost, review rounds, findings, wall time, and whether the run lands without a hand step.
The run also checks the open faults on the way: a real session with `danger-full-access`, the skill load on a real session, and the extra reminder on an idle wake (`codex-idle-wake-extra-reminder`).
It runs as a darn task with profile `docs`, at most `high` effort for Codex, and only on the operator's go-ahead.
