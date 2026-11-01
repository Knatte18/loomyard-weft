# A darn writer hands per-package batches to Sonnet helpers

`darn-sonnet-helpers` · Note · feature, darn, claude, shuttle

## Seen (orch, 2026-11-01)

The darn writer of `gate-one-binary-per-package` reached about 274k tokens of context doing a change across about 1650 test files alone.
The operator wanted it to hand per-package fixes to cheaper Sonnet subagents, but `claudeengine` denies the Agent tool in every non-fork run by a PreToolUse hook (`steerAgentDeny`, `noticeAgentDeny` in `settings.go`), on the reasoning that all work must stay visible in the run's own pane; `Spec.AllowAgentTool` exists but no darn run sets it.

## Idea (operator, 2026-11-01)

- Subagents are visible enough: Claude Code shows them in the session's own pane, and lym already reads fork transcripts under the session's `subagents` directory for activity and usage limits.
  What must stay out of sight-proof runs is a headless `claude -p` launch, not an in-process subagent.
- A darn writer may use the Agent tool to hand batches of per-package work to Sonnet subagents, so a wide change does not run every fix through one large Opus context.
- The deny and its system-prompt notice change to match: the darn writer gets the Agent tool, with the stencil telling it when to use Sonnet subagents (batches of packages, each owning its packages, launched in one message, with the gate rule and its timeout) and when to do the work itself.
- Whether other roles get the same allowance, and whether the hook's visibility reasoning still holds anywhere, is weighed in the same task.
