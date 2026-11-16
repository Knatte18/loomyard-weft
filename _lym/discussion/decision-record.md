## Goal

```markdown
## Seen (adopt-build beam orch, 2026-11-16; the operator saw it)

The board README's entry cell shows a task's brief and one code-span bullet per `run_settings` leaf, and nothing else a run reads.
`internal/boardengine/render.go` `entryCell` renders `Brief` and `settingLines(t.RunSettings)` only.
So `adopt-build-revision`, which has `pull_request` on, shows only `loom: pr_review_ask_parent true`,
and the operator cannot see from the README that the run opens a pull request.

## Idea

Render `pull_request`, and every other per-task field that changes how the run goes and that the cell omits today (such as `profile` and `recipe`), as bullets beside the run settings.
Check the task's field list for others of the same kind.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - run-field-key-literals (class design, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
