# The board README shows every per-task field that changes the run, pull_request included

`readme-shows-run-fields` · Task · bug, board · running · Describe

## Seen (adopt-build beam orch, 2026-11-16; the operator saw it)

The board README's entry cell shows a task's brief and one code-span bullet per `run_settings` leaf, and nothing else a run reads.
`internal/boardengine/render.go` `entryCell` renders `Brief` and `settingLines(t.RunSettings)` only.
So `adopt-build-revision`, which has `pull_request` on, shows only `loom: pr_review_ask_parent true`,
and the operator cannot see from the README that the run opens a pull request.

## Idea

Render `pull_request`, and every other per-task field that changes how the run goes and that the cell omits today (such as `profile` and `recipe`), as bullets beside the run settings.
Check the task's field list for others of the same kind.
