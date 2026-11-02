```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change the diff makes, I list what it touches outside its own lines and whether the diff covers it.

**Per change**

1. **New `subagent_model` loom.yaml key (`config.go`, `template.yaml`).**
   - Covered:
     - Classified in `ConfigSpec` as `none` (`config.go:554`), as `PATTERN-config-layers` requires.
     - Empty value refused in `LoadConfig`, with a way forward; `config_test.go:551` reaches the refusal.
     - `templateConfig()` in `config_test.go:102` updated.
     - A hub whose `loom.yaml` lacks the key takes the template default through the strict `Load` template fill.
     - `formerTemplateValues` (`internal/configsync/configsync.go`) lists only replaced units, so the key needs no entry.
     - The `docs/overview.md` loom-config sentence now names the key.
   - Not covered by the diff: nothing found.

2. **`{{.subagent_model}}` marker in the plan, discussion and darn-write stencils.**
   - Covered:
     - Each stencil's header comment declares the marker.
     - The only fillers of these stencils are `PlanTable`, `DiscussionTable` (via the chair that includes it) and `DarnTable`.
     - `PartsPlanTable` and `internal/testkit/envkit/envkit.go:396` do not fill the plan stencil.
     - No literal `model: "sonnet"` is left in `contracts`, `internal` or `docs`.
   - Not covered by the diff: nothing found.

3. **`darnengine.DarnInputs.SubagentModel` and `wireDarn`'s new parameter.**
   - Covered:
     - The only call site of `wireDarn` is `wiring.go:916`.
     - The only `DarnInputs{` literals are `darnwiring.go:239` and `spec_test.go:25`; both are updated.
     - The value comes from the already-loaded `loomCfg`, so there is no new config read and no layering breach.
     - `DarnTable` refuses an empty `SubagentModel`, and `spec_test.go:199` covers that.
   - Not covered by the diff: nothing found.

4. **`Config.SubagentModel` consumed by `PlanTable` and `DiscussionTable`.**
   - Covered: the test fixtures that build a `Config` (`plan_test.go:42`, `discussion_test.go:52`, `discussiontable_test.go:74`) set the field. Wiring callers pass the loaded `loomCfg`.
   - Not covered by the diff: nothing found.

5. **Rule documents.**
   - `PATTERN-refusal-way-forward`: the new refusal names `lym config loom --set subagent_model=sonnet`.
   - `PATTERN-stencil-ownership` and `PATTERN-no-hard-coded-choice`: the model is read from config.
   - The diff declares no PATTERN changes, and none are needed.
   - Not covered by the diff: nothing found.

6. **Commit messages.** Neither commit carries a `Pattern-Departure:` trailer, and none is needed.

**Findings:** none.

**Tool calls (3):**
1. `grep` for callers of `PlanTable`, `DiscussionTable`, `DarnTable`, `wireDarn` and `DarnInputs{`, plus doc sites of the touched keys and stencils. My context lacked the caller sets.
2. `grep` for other fillers of the changed stencils, `discussion_lenses` in `configsync` and `hubreconcile`, and leftover `"sonnet"`. My context lacked the filler set.
3. Read `envkit.go:380-410` and `configsync.go:40-110`, to confirm the fake plan table fills no marker and `formerTemplateValues` needs no entry.
