# Report: tool schemas in a spawned role's start, by tool and by role

Adds `docs/research/tokens/role-tool-schemas-2026-11-07/` and its line in `docs/research/README.md`.
No code or config of the repository changes; the operator decides which tools to drop from the report's tables.

## What the report finds

- The 21,947 tokens of tool schemas in a role's 29,087-token first request were measured in the sandbox hub `lyt`.
  A role of this hub carries 5,382 tokens of tool schemas in a 13,449-token first request,
  because the tracked `.claude/settings.json` has denied `Artifact`, `ReportFindings`, `ScheduleWakeup` and `SendFeedback` since 2026-10-02.
  Those four weigh 16,755 tokens in a role that is not denied them.
- A role's schemas are not more than twice the orch's.
  The earlier orch figure of 9,927 tokens left out the orch's seven `claude-artifacts` connector tools, 6,208 tokens.
  Counted alike the two are 21,948 and 16,135, a ratio of 1.36, and the gap is the built-in `Artifact` tool.
- `--disallowedTools` on the launch line and `permissions.deny` in the settings both remove a tool's schema from the request.
  Dropping `ScheduleWakeup` cut the first request from 29,087 to 27,461 and 27,465 tokens.
- No session called `Artifact`, `ReportFindings`, `ScheduleWakeup`, `SendFeedback` or `advisor`, over every named session transcript on this machine.
  The per-role table lists, for each role of ten or more sessions, the tools it never called and the 510 to 3,735 tokens its start would lose.

## What is added

- `report.md`: the per-tool schema tokens, the launch comparison, the call counts and the per-role tables.
- `tool-tokens.sh`: prints one request's tool schemas with their tokens, on `measure.sh`'s footing.
- `calls-by-role.sh`: counts, per role, the tools its sessions carried and called.

## Design decisions

- The two launches used a wrapper script as the sandbox hub's `claude` binary, since lym has no setting that drops a tool.
  The hub's config is restored and the wrapper is removed.
- The per-role tables are one matrix, a row per role, instead of one table per role: the schema tokens are the same for every role.
- A tool can go for a role only when a role of ten or more sessions never called it; the report gives no verdict for the other roles.
- The report states, as untested, whether dropping `Agent` or `ToolSearch` also removes the agent-types listing or the deferred stubs.
