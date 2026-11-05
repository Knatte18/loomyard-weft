# Codex trial: one darn run with a Codex writer, end to end, in the cs hub

`codex-trial-darn-e2e` · Task · research, codex, shuttle · draft

## Goal (operator, 2026-11-05)

Part 7's end-to-end case of this project: one darn run with a Codex writer seat, start to landing, in the C# sandbox hub `/home/knatte/Code/lym-csharp-LYMHUB` (repository `Knatte18/lym-csharp`).
Case 6 of `docs/research/codex-engine-trial-pane-load-2026-11-03.md` was not run because a task session cannot clone a hub; the cs hub already exists, so none is cloned.

## Method

- Read the cs repository's code and file one small, real darn task on the cs hub's board (`env -C <cs prime> lym board upsert …`, kind task, recipe darn, profile light), so the run lands real code.
- Give only that entry a Codex writer, `luna[medium]` (effort `high` at most), through the entry's own `run_settings`; never change the cs hub's config, so its other runs stay on Claude.
  Find the key from `lym board upsert --help`, `lym board profiles` and `internal/runprofile/template.yaml`.
  Reviewers and fixers stay on Claude.
- Release it on the cs board and watch it to its landing or a stop, reading `lym loom status`, `lym reed list` and the panes; poll every few minutes, not faster.
  A stop that needs a decision is reported to this run's parent with `lym creel post --parent`, not decided.
- Watch above all: the Codex writer's launch, trust and sandbox; its hooks reaching the run's events; idle and turn-end readings; creel mail and notices reaching the Codex strand and its replies; the writer's commits; and the hand-over to the Claude reviewers and Finalize.

## Deliverables

- A report `docs/research/codex-engine-trial-darn-e2e-2026-11-05.md`: setup, timeline, each check matched or not against `internal/shuttleengine/codexengine/doc.go` and `internal/creelengine/doc.go`, cost and turns.
- Each lym fault as a bug note of its own on this board, in project `codex-engine-trial`, labels `bug`, `codex` and the modules.
- Nothing in the cs repository beyond the trial task's own landing.
