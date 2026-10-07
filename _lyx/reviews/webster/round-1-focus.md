---
round: 1
focus:
  - "Card 1 commit touches `internal/shuttleengine/wait_test.go` and `internal/testkit/envkit/envkit.go`, neither named in the card's Edit list: settle whether each change is needed by the card's behavior and whether it widens scope."
  - "`LoadConfig` and `ConfigOpenMaps`: settle that the map is carried whole when present (an `orch`-only map survives, `driver` is not restored) and that `LoadConfig` adds no TTL validation, per card 1 and `PATTERN-shuttle-provider-seam`."
  - "`claudeengine/command.go`: settle that `resolvePromptCacheTTL` validates the default and every map value in sorted key order, rejects empty, `1H` and `5`, matches `Spec.Role` exactly, and that the error names the key, the value, `5m`, `1h` and `shuttle.yaml`."
  - "`buildLaunchCmd` and `buildResumeCmd`: settle that the TTL wrap sits after the fork-subagent wrap and before the `CLAUDE_ENV_FILE` wrap on both lines, and that `Prepare` resolves once, before any artifact is written, and passes the one value to both builders."
  - "Provider seam: settle that `CLAUDE_CODE_PROMPT_CACHE_TTL` and the `5m|1h` vocabulary appear only in `claudeengine`, and that `shuttleengine` and `configreg` hold no interpretation of the values."
  - "Fixtures: settle that every `Config` handed to `Prepare` takes its TTL fields from the template helper, including `settings_test.go`, `envfile_integration_test.go` and `webstercli/smoke_test.go`, and that no fixture restates the template values."
  - "Test economy: settle for each new test (`TestLoadConfig_PromptCacheTTLKeys`, `TestPrepare_LogsResolvedPromptCacheTTL`, new `TestPrepare_RejectedBeforeArtifacts` and `TestPrepare_ThreadsSpecIntoLaunchCmds` rows) whether it duplicates a named existing test, and whether any added behavior is uncovered."
  - "Per-card mechanical checks and docs: settle that the `test-budget.yaml` rows match the new test-function counts, and that `claudeengine/doc.go`, `shuttleengine/doc.go` and `docs/overview.md` state the behavior the code implements, including that `FORCE_PROMPT_CACHING_5M` still wins and the subagent bucket is untouched."
  - "Comment conventions: settle that new doc comments on `promptCacheTTLEnvKey`, `resolvePromptCacheTTL`, the builders, `ConfigOpenMaps` and the `Config` fields follow `CLAUDE.md`, `scribe:prose` semantic line breaks and the surrounding Go doc style."
---

Seed focus drawn from the plan's three cards against the three commits on the branch.
