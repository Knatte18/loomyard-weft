---
format: 5
approved: true
language: go
---

# Plan: orch live-ready

Make `lyx orch` able to host the real hub orchestrator and prove it against a live Claude Code install.
Shuttle's `Spec` gains a per-run permission mode, an Agent-tool allowance and a resume session id, all realized by `claudeengine` alone;
every shuttle-launched Claude session gets prompt suggestions switched off so the idle probe can pass.
orch then launches in bypass mode with subagents and forks allowed, adopts an already-running session through `lyx orch start --adopt`, and cycles at a natural break below the existing hard cap through a deferrable soft trigger.
The orch smoke suite is extended, run live, and its outcomes recorded in the orchengine package doc.

## Card Index

1 — spec-permission-mode — Spec.PermissionMode realized on claudeengine's launch and resume lines
2 — spec-allow-agent-tool — Spec.AllowAgentTool removes the Agent deny and its notice for one run
3 — prompt-suggestion-off — every generated settings file switches prompt suggestions off
4 — spec-resume-session — Spec.ResumeSessionID launches an existing session with --resume
5 — resume-preflight-seam — optional SessionResumer capability checked by Runner.start before a resume
6 — claude-resume-check — claudeengine's resume check over the transcript and Claude's session registry
7 — orch-config-keys — permission_mode, soft_threshold_tokens and soft_idle_s in orch.yaml
8 — orch-stencils — the adopt launch prompt and the soft handoff request stencils
9 — soft-trigger-watcher — the watcher's soft trigger, DEFER handling and recorded trigger
10 — status-trigger-report — orch status reports the soft threshold, trigger and last deferral
11 — orch-spec-bypass-agents — the orch run carries the configured permission mode, the Agent allowance and forks
12 — orch-start-adopt — lyx orch start --adopt resumes an existing session as the orch strand
13 — orch-live-smoke — extend and run the live orch smoke suite and record its outcomes

## Shared Decisions

### Decision: engine-vocabulary-fields

- **Decision:** `PermissionMode`, `AllowAgentTool` and `ResumeSessionID` are engine vocabulary on `Spec`, like `Effort` and `ForkSubagents`: `Spec.validate` never inspects them, and `claudeengine` is the sole validator and realizer.
- **Rationale:** the Shuttle Provider-Seam Invariant keeps every Claude flag, settings key, registry and transcript fact inside `claudeengine`.
- **Applies to:** cards 1, 2, 4, 5, 6

### Decision: orch-only-caller

- **Decision:** orch is the only production caller that sets any of the new `Spec` fields; no `lyx shuttle run` flag and no shuttle config key reaches them, and the Agent allowance has no `orch.yaml` key.
- **Rationale:** the decision record scopes the knobs to hosting the hub.
- **Applies to:** cards 1, 2, 4, 11, 12

## verify:

go build ./...
go test -tags integration ./internal/shuttleengine/... ./internal/orchengine/... ./internal/orchcli/... ./contracts/stencils/... ./cmd/lyx/...
go vet -tags smoke ./internal/shuttleengine/... ./internal/orchengine/... ./internal/orchcli/...
