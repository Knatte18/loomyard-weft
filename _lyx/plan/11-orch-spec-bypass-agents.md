# Card 11 — orch-spec-bypass-agents

**Edit:**
- `internal/orchcli/start.go`
- `internal/orchcli/cli.go`
- `internal/orchcli/cli_test.go`
- `internal/orchcli/start_test.go`
- `internal/orchengine/doc.go`

**Uses:**
- `internal/orchengine/config.go`
- `internal/shuttleengine/spec.go`

**Intent:**
The orch run's spec carries the permission mode and the Agent-tool allowance, so the hosted hub runs the way the hand-started one does.
`orchSpec` sets `PermissionMode` from `c.cfg.PermissionMode` verbatim, and sets `AllowAgentTool` and `ForkSubagents` both true, so the session can spawn typed subagents and forks while the fork-context `lyx webster` guard stays installed.

Remove `orchShuttleConfig` and its call in `PersistentPreRunE`, together with `TestOrchShuttleConfig_ClearsOnlyAgentDeny`:
the per-run allowance replaces the config rewrite, so orch now loads `shuttle.yaml` unmodified.
Its only caller is in the same file, so the removal and the caller's edit land together.

Bound: under the template's `bypass`, the orch session and every subagent and fork it spawns run every tool with no permission prompt.
Only the orch run sets the allowance, it has no `orch.yaml` switch, and the operator's lever is `permission_mode: prompt`, under which those agents prompt in the orch pane.
An adopted session (card 12) comes back under this same mode whatever mode it ran in before.

Tests: `TestStart_SpecShape` asserts `PermissionMode` equals the configured value, with a harness config of `bypass` and of `prompt`, and `AllowAgentTool` and `ForkSubagents` true.

In the orchengine package doc, add a section naming the permission mode (template `bypass`, passed verbatim, validated by the engine), the Agent-tool allowance with forks, and their bound.

**ImpactSummary:** `orchShuttleConfig` has one caller (`cli.go`'s `PersistentPreRunE`) and one test, both in orchcli; `orchSpec` is the only spec orch's runner ever starts, so every Claude run orch launches still gets the Agent tool, and no other module's runs or config change.
