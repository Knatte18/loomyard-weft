# Card 1 — tapestry-split-keys

**Edit:**
- `internal/loomengine/tapestry.yaml`
- `internal/loomengine#TapestryConfig`
- `internal/loomengine#Config`
- `internal/loomengine#Config.WithTapestry`
- `internal/loomengine#TapestryConfigSpec`
- `internal/loomengine#LoadTapestryConfig`
- `internal/loomengine#TestLoadTapestryConfig`

**Create:**
- `plan:internal/loomengine#Config.PlanTimeout` -> `func (c Config) PlanTimeout(runTimeout time.Duration) time.Duration`
- `plan:internal/loomengine#Config.PartsChairTimeout` -> `func (c Config) PartsChairTimeout(runTimeout time.Duration) (time.Duration, error)`

**Uses:**
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`

**Intent:** Add the plan-split keys of D2 to the tapestry config module, so the two new producers and the row switch read them.

`internal/loomengine/tapestry.yaml` gains these lines after `plan_lenses`, each a hub-tunable key a profile's tapestry block or a task's `run_settings` may set:

```yaml
plan_split: false  # true turns the Parts-Write and Parts-Plan-Write rows on and Plan-Write off, so the plan is written in parts by one writer each and joined by a chair; false, the default, runs Plan-Write alone
parts_depth: light  # reading depth of the Parts-Write agent: light reads the decision record, relevant.md and lym quarry glyphs over the packages the record names; medium adds lym quarry toc of each package the task touches and lym quarry expand of the types the record names
parts_max: 6  # the most parts a parts file may hold; the parts check refuses more
parts: opus[medium]  # model-spec for the Parts-Write agent (see contracts/specs/llm-model-spec.md)
part_writer: opus[medium]  # model-spec for every part writer of the Parts-Plan-Write row
parts_chair: opus[medium]  # model-spec for the chair of the Parts-Plan-Write row, which joins the parts
parts_chair_timeout_min: ~  # minutes the Parts-Plan-Write chair's run may take; null takes twice plan_timeout_min, and a set value must be above plan_timeout_min, since the chair spans every writer
```

`TapestryConfig` gains the fields `PlanSplit bool` (`plan_split`), `PartsDepth string` (`parts_depth`), `PartsMax int` (`parts_max`), `Parts string` (`parts`), `PartWriter string` (`part_writer`), `PartsChair string` (`parts_chair`) and `PartsChairTimeoutMin *int` (`parts_chair_timeout_min`), each with a doc comment; `Config` gains the same seven fields tagged `yaml:"-"`, and `WithTapestry` copies all seven.
`TapestryConfigSpec` gives each of the seven keys `configengine.CategoryNone`: none of them hollows a review or approval gate.

`LoadTapestryConfig` refuses, each naming the key in the existing `tapestry config key %q: …` shape:
- a `parts`, `part_writer` or `parts_chair` value `modelspec.Parse` rejects;
- a `parts_depth` other than `light` or `medium`, the message naming both legal values;
- a `parts_max` below 1, with the way forward to set it to 1 or more;
- a set `parts_chair_timeout_min` not above a non-zero `plan_timeout_min`, the message naming both values and the way forward to set it above `plan_timeout_min` or to null for twice it.

A `plan_timeout_min` of 0 defers to shuttle's `run_timeout_min`, which tapestry.yaml's load cannot read, so the effective timeouts are resolved where the table is built, from the `runTimeout` the caller tells (shuttle's `run_timeout_min` as a duration):
- `Config.PlanTimeout` returns the run's effective plan timeout: `PlanTimeoutMin` minutes, or `runTimeout` when `PlanTimeoutMin` is 0.
- `Config.PartsChairTimeout` returns the chair's timeout: the set `PartsChairTimeoutMin` minutes, or twice `PlanTimeout(runTimeout)` when it is nil.
  A set value not above `PlanTimeout(runTimeout)` is an error naming `parts_chair_timeout_min`, both values and the same way forward; with a non-zero `plan_timeout_min` the load already refused it, so this error is reached only through a 0 `plan_timeout_min`.

`TestLoadTapestryConfig` gains rows: the template loads the seven defaults (`plan_split` false, `parts_depth` light, `parts_max` 6, the three `opus[medium]` models, a nil chair timeout) and `WithTapestry` applies them; each refusal above is reached by a seeded hub value; a set chair timeout with a 0 `plan_timeout_min` loads.
It also gains rows for the two accessors over a fixed `runTimeout`: `PlanTimeout` gives `plan_timeout_min` when set and `runTimeout` when 0; an unset chair timeout gives twice `plan_timeout_min`, and twice `runTimeout` when `plan_timeout_min` is 0; a set one above the effective plan timeout gives that value; a set one not above `runTimeout` with a 0 `plan_timeout_min` is the error.
`TestConfigTemplate_ContainsEveryConfigYAMLTag` already reflects over both structs and needs only the template lines above.

**ImpactSummary:** Adds seven tapestry keys with load-time checks; existing keys and their readers are unchanged.
