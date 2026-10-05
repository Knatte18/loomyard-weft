# Card 2 — Retire the `selfreport` config key

**Edit:**
- `internal/loomengine/template.yaml`
- `internal/loomengine/config.go`
- `internal/loomengine/config_test.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/halt_test.go`
- `internal/loomcli/step_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/smoke_bootstrapwiring_test.go`
- `internal/webstercli/wiring_test.go`
- `internal/orchcli/smoke_live_test.go`

**Intent:**
With no filing left, the `selfreport` knob gates nothing (decision D2).

- `internal/loomengine/template.yaml`: delete the `selfreport` line.
- `internal/loomengine/config.go`: delete `Config.Selfreport`.
- `internal/loomcli/arm.go`: both `observeEntry` calls (`loomPreRun` and `observeStepEntry`) take `c.frictionDir != ""` as their enabled argument instead of `c.cfg.Selfreport`, so the observation follows Tier 2's single "off" signal (`internal/friction`'s empty directory); update the comment beside `loomPreRun`'s call.
- `internal/loomengine/config_test.go`: drop the `cfg.Selfreport` assertion from the template test, and replace `TestLoadConfig_SelfreportFalse` with a test that a `loom.yaml` still carrying `selfreport: false` (written through `writeLoomConfigWithKey`) loads with a nil error and every other key unchanged — the acceptance criterion pinning that the loader ignores the retired key.
  Drop `selfreport: true` from the other inline `loom.yaml` fixtures.
- `internal/loomcli/wiring_test.go` and `internal/webstercli/wiring_test.go`: drop `selfreport: true` from the `loom.yaml` literals.
- `internal/loomcli/smoke_test.go` and `internal/orchcli/smoke_live_test.go`: drop the `strings.Replace` of `selfreport: true` and the paragraph of `fastDeadlineLoomConfig`'s comment that justifies it (no GitHub call remains to guard against).
- `internal/loomcli/smoke_bootstrapwiring_test.go`: reword the comment that says the next run "with selfreport on then files a spurious crash-resume GitHub issue" to say it would write a spurious crash-resume note.
- `internal/loomcli/halt_test.go` and `internal/loomcli/step_test.go`: drop the `Selfreport` field from the fixtures' `loomengine.Config` literals; the busy-step test in `step_test.go` sets `frictionDir` to a temp directory instead, so the observation is enabled the same way production enables it.

Live `_lyx/config/loom.yaml` files are not edited; `lyx config reconcile` drops the stale key when the operator runs it.

**ImpactSummary:** removes one `loom.yaml` key and one `Config` field; the entry observation now runs exactly when Tier 2 is on.

**Commit:** 2: loom: retire the selfreport config key
