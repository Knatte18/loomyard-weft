# Carry the Codex darn trial run codex-greet to its landing

`codex-greet-finish` · Task · research, codex · running · Drift-Baseline

## Goal (operator, 2026-11-05)

Carry part 7's end-to-end case of this project to its end: one darn run with a Codex writer seat, start to landing.
The run exists: task `codex-greet` (T5) on the board of the sandbox hub `/home/knatte/Code/lym-test-LYMHUB` (repositories `Knatte18/lym-test` and `Knatte18/lym-test-weft`), with a `luna[medium]` writer.
It stopped blocked at Darn-Write because the Codex `workspace-write` sandbox could not write the linked worktree's git dir or `~/.cache/go-build`; `docs/research/codex-engine-trial-darn-e2e-2026-11-05.md` reports that first attempt.
This task's dependency `codex-writer-cannot-commit` fixes that, and its landing deploys the fixed binary.

## Method

- Check that the installed `lym` carries the fix: the codexengine writable roots for the worktree's git dir, the common git dir and a writable `GOCACHE`.
- Resume `codex-greet` with `env -C /home/knatte/Code/lym-test-LYMHUB/codex-greet lym loom resume`, or the way forward its refusal names; never cancel or re-create the run.
- Watch the run to its landing or a stop, reading `lym loom status` there, `lym reed list` and the panes; poll every few minutes, not faster.
  A stop that needs a decision is reported to this run's parent with `lym creel post --parent`, not decided.
- Read the run's records afterwards: its events, the Codex rollout under `~/.codex/sessions`, its trace logs and its commits.
- Watch above all: the writer's commits and test runs inside the sandbox, the hand-over to the reviewers, Describe and Finalize.

## Deliverables

- A second section in `docs/research/codex-engine-trial-darn-e2e-2026-11-05.md` for the resumed run: timeline, each check matched or not against `internal/shuttleengine/codexengine/doc.go` and `internal/creelengine/doc.go`, cost and turns.
- Each lym fault as a bug note of its own on this board, in project `codex-engine-trial`, labels `bug`, `codex` and the modules.
