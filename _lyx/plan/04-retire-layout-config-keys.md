# Card 4 — Retire the layout-only reed config keys

**Delete:**
- `internal/reedengine#SelvageConfig`

**Edit:**
- `internal/reedengine#Config`
- `internal/reedengine#ConfigSpec`
- `internal/reedengine#ConfigTemplate`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedengine#newTestEngine`
- `internal/reedengine#TestWithOpLock_PathIsUnderDotLyx`
- `internal/reedengine_test#TestLoadConfig_TemplateDefaultsResolve`
- `internal/reedengine_test#TestLoadConfig_UnknownKeysFromAnOlderConfigAreIgnored`
- `internal/reedengine_test#TestLoadConfig_UninitializedFallsBackToTemplate`
- `internal/reedengine#selvagePaneAllowlist`
- `internal/reedengine#TestSelvageIdentifiersConfinedToSelvagePane`
- `internal/configreg_test#TestFill`
- `internal/configsync#TestReconcileAll`

**Intent:** D5: `collapsed_rows`, `min_full_rows` and the `selvage` block with its `height_rows` go, since card 2 left nothing reading them.

`Config` loses `CollapsedRows`, `MinFullRows` and `Selvage`, and `SelvageConfig` goes.
In `ConfigSpec`, the three keys leave `Categories` and join `RetiredKeys` as `"collapsed_rows"`, `"min_full_rows"` and `"selvage"`, each `configengine.CategoryNone`, which is what admits them in any layer and drops them on load (PATTERN-config-layers, PATTERN-config-strictness: no local workaround).
Both templates lose the three keys; `width` and `height` stay, their comments naming only the unattached session size (`new-session -x`/`-y`) with the fallback-render-box and stack wording dropped.
`ConfigTemplate`'s doc comment stops listing the retired keys.

Tests: `newTestEngine` and `TestWithOpLock_PathIsUnderDotLyx` stop setting the removed fields.
`TestLoadConfig_TemplateDefaultsResolve` drops its two row assertions, and `TestLoadConfig_UninitializedFallsBackToTemplate` drops its `CollapsedRows` and `Selvage.HeightRows` assertions, keeping `Width`.
With `SelvageConfig` and `Config.Selvage` gone, `config.go` carries no Selvage-naming identifier, so `selvagePaneAllowlist` drops its `config.go` entry, `TestSelvageIdentifiersConfinedToSelvagePane`'s doc and failure message name only `selvagepane.go` and `state.go`, and the `selvagepane_enforcement_test.go` header drops the `config.go` carve-out and its note that render exports a `Selvage` type, which card 2 removed.
`TestLoadConfig_UnknownKeysFromAnOlderConfigAreIgnored` replaces its `Selvage.HeightRows` assertion with `Width`, and gains a row whose worktree file holds the template plus `collapsed_rows: 3`, `min_full_rows: 3` and `selvage:\n  height_rows: 1`, which loads and whose `configengine.LoadOrTemplate(dirs, reedengine.ConfigSpec())` merged document names none of the three keys.
`TestFill`'s nested-key row drops `segment_colors.coordinator` instead of `selvage.height_rows`, since the template no longer holds a `selvage` block.
`TestReconcileAll`'s stale-header row, renamed from "stale reed header block is replaced by selvage" to "stale reed header block is removed", no longer expects `selvage.height_rows` added, and a new row seeds a reed file holding the three retired keys and asserts the apply removes `collapsed_rows`, `min_full_rows` and `selvage.height_rows` and leaves no `selvage:` in the file, which is the `lyx config reconcile` prune.

**ImpactSummary:** removes three config fields read nowhere since card 2; the cross-package effects are two config fixture rows, in `internal/configreg` and `internal/configsync`, that named the reed template's `selvage` block.

**Commit:** 4: reed: retire the layout-only config keys
