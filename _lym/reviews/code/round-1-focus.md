---
round: 1
focus:
  - "internal/shellguard/rows.go: does the headless-claude row refuse every `claude -p` / `--print` launch form the Goal names (combined short flags, flag after other args, wrapped in env/sudo/bash -c, pipes and `&&` chains), and let plain interactive `claude` and unrelated `-p` flags through?"
  - "internal/shuttleengine/claudeengine/settings.go and config.go: is every trace of the Agent deny, its notices, `deny_agent_tool` and `Spec.AllowAgentTool` gone, with `steerWebsterForkDeny` intact, and does a config layer still holding the retired key load without error?"
  - "Callers of the removed symbols (orchcli/start.go, loomengine/plan.go, configreg, configsync, template.yaml, shuttleengine/spec.go): does anything still read, write or document the old key or field, and was anything beyond what only served the Agent deny removed?"
  - "contracts/stencils/darn/darn-template-write.md and loom/loom-template-plan.md: do they say fresh Sonnet subagent for lookups and independent per-package batches, each owning its files, launched together in one message, with the gate rule and its timeout, and keep overview work in the session; does any existing stencil rule get dropped or weakened?"
  - "Docs and system-prompt notices (docs/overview.md, doc.go files, pattern/PATTERN-interactive-spawn.md): do they describe the new rule and no longer the old Agent deny?"
  - "Test economy in settings_test.go, prepare_test.go, shuttlecli/smoke_guardrail_test.go, shellguard rows_integration_test.go and burlerengine/smoke_cluster_test.go: is the new headless-claude behavior covered, and does any new test duplicate an existing one?"
---

Seed focus from the Goal in the decision record, which is the only answer key; the plan is absent.
The summary at `_lym/landing/summary.md` was not consulted.
