# Support log — codex-twill-spike

## Interview

A darn run holds no interview.

## Rejected alternatives

- Use `codex exec` for the fan: rejected, the task asks for the interactive TUI as in the first spike, and the engine would drive that.
- Share the operator's `~/.codex` home: rejected, it would hold the operator's allow rules and risk edits to their config.
- Configure hooks to watch the subagents: rejected as the default, since the first spike found none fire for a subagent; rollouts are the audit source.

## Review rounds

_No rounds yet._

## Question ledger

- Which model and effort: self-pick `gpt-6-luna` at `low`, the cheapest, as in the first spike.
- How many subagents: self-pick four, the top of the task's range, so a parallelism limit shows if one exists.
- How many `fork_turns` tries: self-pick at least three fresh sessions, more if the account's limits allow.
