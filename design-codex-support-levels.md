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
   Since commit `9874c966` the template default is `sandbox_mode: auto`: a launch probes the sandbox and runs `danger-full-access` itself, with a warning, on a machine whose bubblewrap cannot start, so the bench needs no hand-set mode.
   That probe path is also checked by unit tests only, until a real session runs it.
3. **The rest of the tapestry recipe.** Codex in Discussion, Plan, Plan-review and Code-Review as the main seat, and in raddle.
   Not tried; it waits until level 2 shows the quality holds.

## First work: the level 2 bench

A bench of Codex as the darn writer against Claude on one or two real darn tasks, each run once per writer.
It reads, per run: cost, review rounds, findings, wall time, and whether the run lands without a hand step.
The run also checks the open faults on the way: a real session launched under `auto` on a machine where bwrap fails (the probe, the warning and a `danger-full-access` run), the skill load on a real session, and the extra reminder on an idle wake (`codex-idle-wake-extra-reminder`).
It runs as a darn task with profile `docs`, at most `high` effort for Codex, and only on the operator's go-ahead.

## Built since (tidy, 2026-11-15)

Task `codex-writer-bench-neve` (`c0815831`) ran the first bench run on Neve: `astra[high]` as the darn writer, replayed on the landed `ide-beam-readme-preview` (`824eb2b6`) through a bare `lym shuttle run`.
Report: `docs/research/codex/darn-writer/codex-darn-writer-replay-neve-2026-11-15/`.

- Its tree passes the gate on `./internal/ideconfig`, and its `template.yaml` and `config_test.go` match the landed change byte for byte.
- It added a documentation page the entry did not ask for, and its change description names no design decisions.
- About 4 minutes against 5 minutes 41 seconds for the landed writer, whose figure includes a 2 minute 17 second verify gate; dollar cost unknown.
- The session ran in a scratch worktree, so no board, drift or loom check ran, and it was not a whole darn run: the "lands with no resumption" test of level 2 is still open.
- It got no skills, no guards and Claude-worded stencils; see `codex-same-as-claude`.
  The operator judges level 2 after that note's fix and a rerun of the bench, and its OSL twin `codex-writer-bench-osl`.
