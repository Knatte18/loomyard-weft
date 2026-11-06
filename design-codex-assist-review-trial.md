# Codex as assistant Code-Review reviewer on real darn runs

`codex-assist-review-trial` · Note · research, codex, shuttle

## Decision (operator, 2026-11-06)

Codex is used for review first, or as a support reviewer, not as a writer: the operator does not trust that it writes in the same tone as Claude.
The earlier idea of a writer comparison is dropped.

## Idea

`loom.yaml` already has `code_assist_reviewer_model`: an extra reviewer beside the Code-Review reviewer, whose report the reviewer consolidates and which never gates the round.
Set it per task through `run_settings`, as `{"loom":{"code_assist_reviewer_model":"astra[medium]"}}`, on real darn runs, and read afterwards what Codex found that the Claude reviewer did not, its false findings, and the time it added.

The benches `docs/research/codex/review/codex-code-review-bench-2026-10-28` and `codex-review-yardstick-2026-10-28` found Codex no stand-in for the Claude fan but with real findings of its own, which fits a support role.

First input proposed: the Webster bug B7 `begin-batch-wrong-conflict-kind`, as darn light.
