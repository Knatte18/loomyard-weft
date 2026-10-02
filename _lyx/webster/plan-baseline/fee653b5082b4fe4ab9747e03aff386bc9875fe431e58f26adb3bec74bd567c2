# Card 7 — orch-config-keys

**Edit:**
- `internal/orchengine/config.go`
- `internal/orchengine/template.yaml`
- `internal/orchengine/config_test.go`

**Create:**
- `internal/orchengine#Config.SoftThreshold`
- `internal/orchengine#Config.SoftIdle`

**Intent:**
Add three `orch.yaml` keys through the embedded template, per the Config Strictness Invariant:

- `permission_mode`, template value `bypass`: passed to the orch run's `Spec.PermissionMode` verbatim.
  orch never validates it, so the engine stays the single validator; the comment names `bypass` and `prompt` and says `prompt` makes the session and every subagent it spawns prompt in the orch pane.
- `soft_threshold_tokens`, template value 300000: the context reading at which the watcher may cycle at a natural break, below `threshold_tokens`, which stays the hard cap.
  The comment says a value at or above `threshold_tokens` never fires.
- `soft_idle_s`, template value 300: how long the newest turn end must be quiet before a soft cycle, and how long a `DEFER` reply holds the next soft attempt.

`Config` gains `PermissionMode`, `SoftThresholdTokens` and `SoftIdleS`, and `SoftThreshold` and `SoftIdle` floor a non-positive value to the template default, like the existing accessors, with new `default...` constants beside the others.
Update the `threshold_tokens` comment to call it the hard cap.

A present `orch.yaml` without the new keys fails to load with the existing "run lyx config reconcile" error, so the hub's own `orch.yaml` needs `lyx config reconcile` once this deploys.

Tests: the template loads with `permission_mode` `bypass`, `SoftThreshold` 300000 and `SoftIdle` 300 seconds;
zero and negative values for both soft keys floor to their defaults.

**ImpactSummary:** Every `lyx orch` verb fails to load an existing `orch.yaml` until `lyx config reconcile` adds the three keys, and the in-repo fixtures that seed one (`config_test.go` here, the smoke test in card 13) must carry them; no module outside orch reads this config, and the existing accessors keep their values.
