# Allow subagents and forks in every run; refuse headless claude -p

`claudeengine` used to deny the in-process Agent tool in every run, or let only forks through in fork-mode runs, on the reasoning that all work must stay visible in the run's pane.
That left a darn writer at high context unable to hand per-package work to Sonnet helpers, and a Plan writer unable to hand its lookups to a fresh subagent; a fork inherits the whole context, so it saves nothing.
Claude Code shows a subagent in the session's own pane, and lym reads its transcript under the session's `subagents` directory, so the deny protected nothing.
What does hide work is a headless `claude -p` launch, so that is now what stays refused.

## What changes

- No run denies the Agent tool any more.
  `steerAgentDeny`, `steerAgentNonForkDeny` and their notices are gone from `claudeengine/settings.go`, with `Spec.AllowAgentTool` and the `deny_agent_tool` key in `claude.yaml` and its template.
  The Webster fork guard (`steerWebsterForkDeny`) stays.
- A new `headless-claude` row in `internal/shellguard` refuses a Bash command that launches `claude` or `claude.exe` with `--print` or a single-dash flag word holding `p` (`-p`, `-pc`).
  It is installed in every run, the orch included, with no config key.
  Its steer and notice say the work would show in no pane and name a subagent through the Agent tool as the way forward.
- The darn write stencil and the loom plan stencil say when to hand work to a fresh Sonnet subagent: lookups, and batches of independent per-package work whose edits are already decided, one subagent per package, each owning its files, all launched in one message with the gate rule and its Bash timeout.
  Work that needs the overview of the change stays in the session.
- `docs/overview.md`, the `claudeengine` and `shuttleengine` package docs, the `shellguard` package doc and `PATTERN-interactive-spawn.md` describe the new rule.
  The orch spec keeps `ForkSubagents` and drops `AllowAgentTool`.

## Design decisions

- `deny_agent_tool` is a retired key, not a silently dropped one.
  `configreg` no longer moves the old `shuttle.claude_deny_agent_tool` key into it, and `claudeengine.ConfigSpec` lists it under `RetiredKeys` as a guard-category key.
  A layer holding it is read without it, and a local override that sets it is refused with the way forward "delete the key".
  Hubs that set `deny_agent_tool: false` as a stopgap load unchanged.
- The `headless-claude` row is a guardrail, not a barrier, like its neighbours.
  A launch behind `bash -c`, a script, or an assignment prefix such as `X=1 claude` passes.
  A `claude` subcommand taking a single-dash flag word holding `p` of its own is falsely denied, as is a matching spelling after a separator inside a quote that the command-position pattern misreads.
  These limits are in the row's comment and the package doc.
- The settings golden file shrank by most of its size because the Agent deny rows no longer appear in any matrix.

## To check by hand

- Start an interactive darn run and a Plan run, have each launch a Sonnet subagent through the Agent tool, and confirm it appears in the run's pane and its transcript is read from the `subagents` directory.
- In a run, try `claude -p "x"` and `claude --print x` through Bash and confirm the deny and its notice; try `claude --resume` and confirm it passes.
- Load a hub whose `claude.yaml` still holds `deny_agent_tool`, and confirm `lym` reads it without error and refuses it in a local override.
- The tmux and llm tiers were not run for this change: `internal/burlerengine/smoke_cluster_test.go`, `internal/orchcli/smoke_live_test.go` and `internal/shuttlecli/smoke_guardrail_test.go` changed and should be run on a machine with those tiers.
