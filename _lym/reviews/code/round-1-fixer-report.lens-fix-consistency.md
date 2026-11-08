```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 and F2 change set for agreement with its surroundings.

- **LOW**: the file header comment in `internal/loomcli/wiring_darn_test.go` (lines 1–5) lists what the file covers but doesn't mention the new `TestWire_DriftExpectPlanReading`. The header names `TestWire_DarnRecipe`'s scope in detail and says nothing about the per-recipe Drift plan-reading check. A reader choosing a file by its header would not learn that this test lives here.
- No other findings:
  - `docs/overview.md:1227` and `:1231` agree with the sampler paragraph at `:1014-1016`, which names the same `drift.enabled` and `drift.scope == full` gate and the `Sampler-Session` routing.
  - The rows list at `:1226` is unchanged.
  - The test's assertion on `plan_reading` matches the `"none:"` text and the plan-directory text in `internal/driftengine/seats.go:194-196`.

Further tool calls:
1. `grep -n "drift.scope\|Sampler-Session" docs/overview.md`: to find the other places the overview names the sampler's scope gate; my context only held lines 1215–1244.
2. `Read docs/overview.md` lines 1008–1019: to compare the new sentences against the sampler paragraph; that span was not in my context.
