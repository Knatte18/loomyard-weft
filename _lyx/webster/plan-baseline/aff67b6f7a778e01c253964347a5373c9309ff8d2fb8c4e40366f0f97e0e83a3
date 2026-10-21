# Card 3 — claude-orch-webster-batcher-tuned-defaults

**Edit:**
- `internal/shuttleengine/claudeengine/template.yaml`
- `internal/orchengine/template.yaml`
- `internal/websterengine/template.yaml`
- `internal/batcher/template.yaml`
- `internal/orchengine#defaultThresholdTokens`
- `internal/orchengine#defaultSoftThresholdTokens`
- `internal/orchengine/doc.go`
- `docs/overview.md`
- `internal/shuttleengine/claudeengine#TestLoadConfig`
- `internal/orchengine_test#TestLoadConfig_TemplateResolvesWithNoFile`
- `internal/orchengine_test#TestConfig_AccessorsFloorNonPositive`
- `internal/battencli#TestLoadOverviewThresholds`
- `internal/websterengine_test#TestLoadConfig`
- `internal/batcher_test#TestActive`
- `internal/hubreconcile_test#seedRetiredKey`

**Intent:**
D1's remaining values become the embedded templates' values, each changed key's comment describing the new default:
- `claude`: `prompt_cache_ttl_roles` becomes `driver: 5m`, `webster: 1h`, `orch: 1h`.
- `orch`: `soft_threshold_tokens: 150000` and `threshold_tokens: 200000`; their comments name the new floors.
  `defaultSoftThresholdTokens` and `defaultThresholdTokens`, which the const block names as the template defaults the accessors floor a non-positive value back to, become 150000 and 200000, so a floored value equals the template's again;
  `internal/orchengine/doc.go`'s sentence on the template hard cap and soft threshold states the new numbers and drops "sized for a session with a context window of about one million tokens" unless it still holds for 200000.
- `webster`: `kickstart_pack: false`, its comment naming `lyx config webster --set kickstart_pack=true` as the way to turn it on.
- `batcher`: the `cautious` profile's `budget: 200000`.

Tests that read the template defaults follow them:
- claudeengine's `TestLoadConfig` expects the new roles map where it loads the template's.
- orchengine's `TestLoadConfig_TemplateResolvesWithNoFile` expects 200000 and 150000, and `TestConfig_AccessorsFloorNonPositive` expects a non-positive value to floor to 200000 and 150000.
- battencli's `TestLoadOverviewThresholds` row "no orch.yaml resolves the template thresholds" expects `Soft: 150000` and `Hard: 200000`.
- websterengine's `TestLoadConfig` rows that load the template expect `KickstartPack: false`.
- batcher's `TestActive` row that replaces the template's `budget: 450000` replaces `budget: 200000` instead.
- `seedRetiredKey` in `hubreconcile`'s integration tests replaces `budget: 200000` with `differingBatcherBudget`, whose value stays one that differs from the template's, and its fatal guard names the new line.

The implementer greps the test files for the old values (`kickstart_pack` expected on, `prompt_cache_ttl_roles` with `driver: 1h`, `300000` and `400000` as orch thresholds, `budget: 450000`) and fixes in this card's commit any assertion that reads a template default rather than seeding its own; the files to check first are `internal/websterengine/beginbatch_test.go` (`TestBeginBatch_KickstartPack`'s `kickstart_pack=on` log rows), `internal/shuttleengine/claudeengine/prepare_test.go`, `internal/orchcli`'s `cli_test.go`, `smoke_cycle_test.go` and `smoke_live_test.go`.
`internal/battencli/overview_test.go`'s `orchThresholds` fixture, which hands its own thresholds in, stays as it is.
`docs/overview.md`'s prompt-cache sentence, "(template: 1 hour for `driver` and `webster`, 5 minutes for every other role)", states the new roles map: 5 minutes for `driver`, 1 hour for `webster` and `orch`, 5 minutes for every other role.

**ImpactSummary:** four templates' defaults, two orch floor constants and the orch doc sentence; the test members that read or rewrite those defaults follow them, and the orch watcher floors to the new values.
