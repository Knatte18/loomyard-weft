# A darn writer hands per-package batches to Sonnet helpers

`darn-sonnet-helpers` · Note · feature, darn, claude, shuttle

## Seen (orch, 2026-11-01)

The darn writer of `gate-one-binary-per-package` reached about 274k tokens of context doing a change across about 1650 test files alone.
The operator wanted it to hand per-package fixes to cheaper Sonnet subagents, but `claudeengine` denies the Agent tool in every non-fork run by a PreToolUse hook (`steerAgentDeny`, `noticeAgentDeny` in `settings.go`), so that all work stays visible in the run's own pane; `Spec.AllowAgentTool` exists but no darn run sets it.

## Idea (operator, 2026-11-01)

A darn writer may hand batches of per-package work to Sonnet subagents, so a wide mechanical change does not run every fix through one large Opus context.
The design question is how this fits the rule that every agent's work stays visible in a pane: either allow the Agent tool with model `sonnet` for the darn writer (as `AllowAgentTool` or a fork-like allowance), or launch each batch as its own visible lym session at a Sonnet model.

The second way may need little new code: `lym shuttle run --model <sonnet> --role <role> --prompt-file <file>` starts one agent in its own visible pane and blocks until it ends, and no task-session guard refuses it today (only `lym fabric` verbs, hub-wide config writes and run-settings changes are refused), so a darn writer could launch one per batch as background calls (operator, 2026-11-01).
What is untested: which role such a helper takes and what its stencil tells it, how its gate runs share the slots, how its cost is counted with the run's, and whether the writer waits on several at once without one turn per helper.
