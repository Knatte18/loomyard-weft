# Codex as assistant Code-Review reviewer on real darn runs

`codex-assist-review-trial` · Note · research, codex, shuttle

## Decision (operator, 2026-11-06)

Codex is used for review first, or as a support reviewer, not as a writer: the operator does not trust that it writes in the same tone as Claude.
The earlier idea of a writer comparison is dropped.

## Idea

`loom.yaml` already has `code_assist_reviewer_model`: an extra reviewer beside the Code-Review reviewer, whose report the reviewer consolidates and which never gates the round.
Set it per task through `run_settings`, as `{"loom":{"code_assist_reviewer_model":"astra[medium]"}}`, on real darn runs, and read afterwards what Codex found that the Claude reviewer did not, its false findings, and the time it added.

The benches `docs/research/codex/review/codex-code-review-bench-2026-10-28` and `codex-review-yardstick-2026-10-28` found Codex no stand-in for the Claude fan but with real findings of its own, which fits a support role.

## Results so far

Run 1, `begin-batch-wrong-conflict-kind` (`b93a0a52`), darn light, `astra[medium]` solo, no lenses, beside a `sonnet[medium]` Chair with three lenses, one round:

- Astra's one finding was real and no Claude seat had it: two test readers kept a literal beside the new constant, fixed in the landing.
  Astra filed it BLOCKING; the Chair kept it at LOW.
- Astra had no false finding, but missed both MEDIUM findings the affected-set lens found in the webster master stencil.
  It read the plan fixer's limit and judged the routing fine.
- Astra's review evidence was sound, but its sandbox could not run the gate: the runner lock under `_bolt/.lym/runproc/` was read-only.
- Time: Astra ran 11:22:27 to 11:24:36, in parallel; the Chair ended at 11:25:35, so at most a small wait on the wall clock.
  `round-<n>-usage.yaml` records no time or tokens for the assistant.

One run is no verdict.
