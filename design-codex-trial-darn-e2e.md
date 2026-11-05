# Codex trial: follow the sandbox darn run with a Codex writer to landing and report

`codex-trial-darn-e2e` · Task · research, codex, shuttle · done

## Goal (operator, 2026-11-05)

Part 7's end-to-end case of this project: one darn run with a Codex writer seat, start to landing.
The run already exists: task `codex-greet` (T5) on the board of the sandbox hub `/home/knatte/Code/lym-test-LYMHUB` (repositories `Knatte18/lym-test` and `Knatte18/lym-test-weft`), released by the orch on 2026-11-05 with a Codex writer and running at Darn-Write.
This task follows that run and reports on it; it starts no run of its own and never restarts, cancels or changes `codex-greet`.

## Method

- Find how the run's writer seat was set to Codex (the sandbox hub's config or the entry's settings) and record it.
- Watch the run to its landing or a stop, reading `env -C /home/knatte/Code/lym-test-LYMHUB/codex-greet lym loom status`, `lym reed list` and the panes; poll every few minutes, not faster.
  A stop that needs a decision is reported to this run's parent with `lym creel post --parent`, not decided.
- Read the run's records afterwards: its events, the Codex rollout under `~/.codex/sessions`, its trace logs and its commits.
- Watch above all: the Codex writer's launch, trust and sandbox; its hooks reaching the run's events; idle and turn-end readings; creel mail and notices reaching the Codex strand and its replies; the writer's commits; and the hand-over to the reviewers and Finalize.

## Deliverables

- A report `docs/research/codex-engine-trial-darn-e2e-2026-11-05.md`: setup, timeline, each check matched or not against `internal/shuttleengine/codexengine/doc.go` and `internal/creelengine/doc.go`, cost and turns.
- Each lym fault as a bug note of its own on this board, in project `codex-engine-trial`, labels `bug`, `codex` and the modules.
