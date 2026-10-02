# lyx orch: ready to host the real hub orchestrator

`orch-live-ready` · Planned · feature · running · Webster

Found 2026-09-30:
- orch launches as an interactive shuttle run (internal/orchcli/start.go:61), so no --dangerously-skip-permissions; the hub runs in bypass mode. Want a permission-mode option, bypass as the orch default.
- shuttle denies the Agent tool in every run (claude_deny_agent_tool), so orch cannot spawn review or fix agents; ForkSubagents is false.
- no way to adopt a running session (`--adopt <session-id>` via claude --resume, watcher bound to the existing transcript).
- soft threshold: hand off at a natural break from about 300k, 400k hard cap.
- risk: SessionIdle may read Claude's greyed prompt suggestion as not idle, so no cycle ever runs; verify live.