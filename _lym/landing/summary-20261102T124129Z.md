# Discussion writers hand lookups to subagents; subagent model is config

The Discussion-Write stencil now offers the writer the option to hand a batch of independent lookups to fresh subagents, guidance the Plan stencil also carries.
The model those subagents run on, hardcoded as `model: "sonnet"` in the Plan and darn-write stencils until now, is read from a new loom config key, `subagent_model`.

## What changes

- `contracts/stencils/loom/loom-template-discussion.md`, Step 2: the writer may launch a batch of independent lookups together in one message, each subagent with `subagent_type: "general-purpose"` and the configured model, each prompt carrying its whole question and asking for a short answer naming files and spans.
  The stencil leaves the choice to the writer: subagents pay off for a broad search whose conclusion alone it needs, while a small lookup, or code it needs to see itself, stays in its own session.
  A fork does not serve, since it inherits the whole context.
- `loom.yaml` gains `subagent_model`, defaulting to `sonnet`, so behaviour is unchanged.
  Its template comment names the three stencils that read it.
  It is open to the local layer, and `LoadConfig` refuses an empty value, naming `lym config loom --set subagent_model=sonnet` as the way forward.
- The Plan, Discussion-Write and darn-write stencils render `model: "{{.subagent_model}}"` where they hardcoded `model: "sonnet"`.
  `PlanTable` and `DiscussionTable` fill the marker from `Config.SubagentModel`.
  `DarnInputs` gains a required `SubagentModel`, which `wireDarn` fills from the loaded loom config.
- `docs/overview.md` lists the new key with the other `loom.yaml` keys.

## Design decisions

- **Key name and module.** The key is `subagent_model` in `loom.yaml`, next to the writer keys `discussion` and `discussion_lenses`.
  The darn writer reads it too, even though darn has its own `darn.yaml`, because the decision record puts all three stencils on one key.
- **A bare model name, not a model-spec.** The value goes straight into the Agent tool's `model` field, which takes a name such as `sonnet` and no effort, so it is not parsed or resolved through `models.yaml`.
  Only an empty value is refused.

## Tests

- `TestPlanTable`, `TestDiscussionTable` and `TestDarnSpec` set the key to `haiku` and assert that the rendered prompt carries `model: "haiku"`, which shows the model comes from config and not from a literal.
- `TestLoadConfig_Refuses` has a new row for an empty `subagent_model`, and `templateConfig` expects the template's `sonnet`.
- `TestDarnSpec_Refusals` has a new row for an empty `SubagentModel`.
