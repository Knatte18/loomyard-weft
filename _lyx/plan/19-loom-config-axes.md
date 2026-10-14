# Card 19 — loom-config-axes

**Create:**
- `plan:internal/loomengine#DiscussionAdvisor` -> `type DiscussionAdvisor struct`
- `plan:internal/loomengine#DiscussionAdvisorList` -> `type DiscussionAdvisorList []DiscussionAdvisor`
- `plan:internal/loomengine#DiscussionAdvisorList.UnmarshalYAML` -> `func (l *DiscussionAdvisorList) UnmarshalYAML(node *yaml.Node) error`
- `plan:internal/loomengine#validateDiscussionAdvisors` -> `func validateDiscussionAdvisors(advisors DiscussionAdvisorList, reg modelspec.Registry) error`
- `plan:internal/loomengine#TestMigrateConfig` -> `func TestMigrateConfig(t *testing.T)`
- `plan:internal/loomengine#planLensDirective` -> `func planLensDirective(layout *lyxcwd.Location, dirs configengine.Dirs, stencilsDir, fan string) (string, error)`
- `plan:internal/loomengine#discussionLensDirective` -> `func discussionLensDirective(layout *lyxcwd.Location, dirs configengine.Dirs, stencilsDir, fan string) (string, error)`

**Delete:**
- `internal/loomengine#DiscussionProducerSingle`
- `internal/loomengine#DiscussionProducerSeats`
- `internal/loomengine#DiscussionProducerTwill`
- `internal/loomengine#PlanProducerSingle`
- `internal/loomengine#PlanProducerTwill`
- `internal/loomengine#twillFan`
- `internal/loomengine#twillPlanFan`
- `internal/loomengine#validateAdvisorSpecs`
- `internal/loomengine#planTwillDirective`
- `internal/loomengine#discussionTwillDirective`

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#ConfigOpenMaps`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#MigrateConfig`
- `internal/loomengine#advisorWayForward`
- `internal/loomengine/template.yaml`
- `internal/loomengine#twillLenses` -> `func twillLenses(dirs configengine.Dirs, fan, key string) ([]twilldirective.Lens, error)`
- `internal/loomengine#DiscussionAdvisorNotes` -> `func DiscussionAdvisorNotes(l *lyxcwd.Location, name string) string`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#PlanTable`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestDiscussionPaths`
- `internal/loomengine#discussionTableConfig`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestDiscussionTable_Refuses`
- `internal/loomengine#TestPlanTable`
- `internal/shedrecipe#planWriteGateSpec`
- `internal/loomcli#TestWire_DiscussionTableEvaluatesToExpectedShape`
- `internal/loomcli#TestNewParentReviewConfig_KeepAliveInterval`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestModules_LocalLayerRefusals`
- `internal/boardcli_test#stepRunSettings`
- `//contracts/stencils/loom/loom-template-discussion-chair.md`
- `//contracts/stencils/loom/loom-template-discussion-advisor.md`
- `//contracts/specs/refusal-spec.md`
- `//docs/overview.md`

**Intent:** D7, D8, D9 and D11: Discussion's and Plan's `*_producer` switches become two axes, named advisors and lens forks, and old files migrate.

`loom.yaml` keys (`Config`, `template.yaml`, `ConfigSpec`, and `config.go`'s file header, which lists every key):
- `discussion_producer` and `plan_producer` are retired, with their constants.
- `discussion_advisors` becomes a `DiscussionAdvisorList`, each entry `{name, model}`: `name` the advisor's seat name, `model` its model-spec. Template default `[]`. A guard key in `ConfigSpec`, as `discussion_producer` was, so only the hub-wide file or a worktree's own config sets it; `configengine.ValidateLocal` refuses it in a local layer or a task's `run_settings`. It stays in `ConfigOpenMaps`, carried whole.
- `discussion_lenses` (D8) names the `burler.yaml` fan whose lenses the Discussion lead forks itself for once its record is written and its self-check passes; empty, the template default, turns the forks off. A guard key. Its template comment and the `Config.DiscussionLenses` field's doc comment say it names the writer's lens fan, not the Discussion-Review perch's `discussion_fan`.
- `plan_lenses` (D9) does the same for Plan-Write; template default empty; a guard key. Plan has no advisor key.
Together the three guard keys bound no more than the key they replace: no local override or `run_settings` can start advisors, turn the forks off or change their fan.

Load (`LoadConfig`):
- `DiscussionAdvisorList.UnmarshalYAML` accepts a sequence of mappings with the keys `name` and `model`; a string entry is refused naming the entry and the way forward, "rewrite the entry as {name: <seat name>, model: <model-spec>}" (D11: a sparse worktree file holding string entries and no `discussion_producer` keeps them unmigrated and is refused here, never emptied); any other shape is refused naming the key.
- `validateDiscussionAdvisors` (replacing `validateAdvisorSpecs`) refuses, naming the 1-based entry and ending with `advisorWayForward`: a missing or blank name, a duplicate name, a name `seatengine.ValidateSeatName` refuses (D6's seat-name rule), and a model-spec that does not parse or that the registry does not define. An empty list loads as no advisors, and `models.yaml` is read only when the list is non-empty, as today.
- A file that sets none of `discussion_advisors`, `discussion_lenses` and `plan_lenses` loads with no advisors and no lens forks.

Table build:
- `DiscussionTable` builds one advisor per `cfg.DiscussionAdvisors` entry, whatever else is set, named by the entry's `name` (no `advisor-<n>` name is formed any more) and writing `DiscussionAdvisorNotes(layout, name)`, now `_lyx/discussion/advisor-<name>.md`, so no name takes a lead output's path; the lead forks when `cfg.DiscussionLenses` is set, reading that fan, and the advisors never fork (D8). Advisors with lenses on is the bench's round-4 combination.
- `PlanTable`'s lead forks when `cfg.PlanLenses` is set, reading that fan.
- `twillLenses` takes the fan name and its key in place of the spawning row.
  `discussionLensDirective` and `planLensDirective` replace `discussionTwillDirective` and `planTwillDirective` with the same bodies, taking the fan name and passing `twillLenses` their own key, `discussion_lenses` or `plan_lenses`; the old pair goes rather than being re-signed, since their remaining callers sit in `DiscussionSpec` and `PlanSpec`, which card 18 deletes.
  A name with no fan, or a fan with no lenses, in `burler.yaml` (whose fans fall back per name to the embedded template) is refused at table build naming the key, the fan and `burler.yaml`, with the way forward "define the fan and its lenses in burler.yaml, or set the key to \"\" to turn the lens forks off"; an unreadable `burler.yaml` is refused naming the key.
- `planWriteGateSpec`'s comment names `plan_lenses`.

`MigrateConfig` (reached from `hubreconcile.Ensure` through the module's `Migrate` hook, PATTERN-binary-change-reconcile) keeps the `step_idle_timeout_min` floor and rewrites one present document, a report line per rewrite:
- `discussion_producer: twill` becomes `discussion_lenses: twill-discussion`; `single`, `seats` and any other value become `discussion_lenses: ""`.
- With `discussion_producer: seats`, each string entry n of the document's `discussion_advisors` becomes `{name: advisor-<n>, model: <entry>}`, the report line asking the operator to rename them (the one place `advisor-<n>` survives, until renamed).
- With `discussion_producer: single` or `twill` (or any other value), `discussion_advisors` is set to `[]`, written even when the document set no list, so a lower layer's list stays off where this document turned it off.
- With `discussion_producer: seats` and no `discussion_advisors` of its own, the list is left as it is and the report line says its advisors came from a lower layer whose migration may have emptied them, asking the operator to set its named list.
- `plan_producer: twill` becomes `plan_lenses: twill-plan`; `single`, empty or any other value becomes `plan_lenses: ""`.
- The `discussion_producer`-keyed rewrites of `discussion_advisors` apply only to a document that sets `discussion_producer` itself; a document without either retired key is returned unchanged, so a migrated file is unchanged on a second pass. Comments are carried through on the yaml node tree, as `batcher.MigrateConfig` does.

Every new refusal names its way forward and lands its `contracts/specs/refusal-spec.md` row (PATTERN-refusal-way-forward): the string entry, the blank, duplicate and invalid name, the missing or empty lens fan for each key; the rows for the unknown `discussion_producer` and `plan_producer` values go, and the twill rows and the advisor model-spec rows are reworded to the new keys.
`docs/overview.md`'s Discussion and Plan paragraphs and the two discussion stencils' header comments describe the two axes.

Tests:
- `TestMigrateConfig` rows: each `discussion_producer` value with and without a `discussion_advisors` list, each `plan_producer` value, the report lines, a second pass leaving the bytes unchanged, a sparse document with string entries and no producer left unchanged, and a layered load where a worktree `discussion_producer: twill` over a hub `seats` with advisors, both migrated, loads with no advisors.
- `TestLoadConfig_Loads`: the template loads no advisors and empty lens keys; named advisors and set lens keys load; `templateConfig` and the producer rows follow the fields.
- `TestLoadConfig_Refuses`: a string advisor entry naming the way forward, a blank, duplicate and numeric name, an unresolvable model, and a non-sequence value; the unknown-producer rows go.
- `TestModules_EveryTemplateKeyClassified` lists `discussion_advisors`, `discussion_lenses` and `plan_lenses` among loom's guard keys and drops the producer keys; `TestModules_LocalLayerRefusals` refuses `discussion_advisors` and `discussion_lenses` in a local layer; `stepRunSettings`' closed-key row sets `discussion_advisors` in `run_settings` and wants it refused by name.
- `TestDiscussionTable` and `discussionTableConfig` build from named advisors and `discussion_lenses`: advisors with lenses on fork from the lead only (no advisor sets `ForkSubagents` or carries the twill directive), the advisors' strands read `discussion-<name>` and their notes `advisor-<name>.md`, the template config builds a lone lead with no fork; `TestDiscussionTable_Refuses` and `TestPlanTable` gain a lens key naming a missing fan and a fan with no lenses, and their unparsable-`burler.yaml` rows set the lens key and expect the key's name in the error; `TestDiscussionPaths` reads a named notes path.
- `TestWire_DiscussionTableEvaluatesToExpectedShape` seeds named advisors and lens keys; `TestNewParentReviewConfig_KeepAliveInterval`'s rows drop the producer field.

**ImpactSummary:** Retires two loom.yaml keys and reshapes a third for every hub through the reconcile migration; every reader of the old keys is in loomengine, its tests and the two config-registry tests listed here.
