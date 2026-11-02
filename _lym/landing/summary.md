# Discussion writers hand lookups to subagents; subagent model is config

The Discussion-Write stencil gains the explore-subagent guidance the Plan and darn-write stencils already carry, and the subagent model stops being a literal in each stencil.
Lookups made late in a large writer session are paid at its whole context, and Discussion-Write explores the code more than any other role.

## What changes

- `loom-template-discussion.md` tells the writer it may hand a batch of independent lookups to fresh `general-purpose` subagents, launched together in one message.
  The writer keeps a lookup it judges small, or code it needs to see itself.
  Each prompt carries its whole question and asks for a short answer naming files and spans.
  A fork does not serve, since it inherits the writer's whole context.
- `loom.yaml` gets a `subagent_model` key, defaulting to `sonnet` in `internal/loomengine/template.yaml`, so behaviour is unchanged.
  Its template comment names the three stencils that read it.
- `loom-template-plan.md`, `loom-template-discussion.md` and `darn-template-write.md` render `model: "{{.subagent_model}}"` in place of the hardcoded `sonnet`.
- `docs/overview.md` lists the key among loom's config keys.

## Design decisions

- The key lives in `loom.yaml` beside the writer-model keys, since all three writers belong to loom recipes.
  It is a bare model name the Agent tool takes, not a bracketed model-spec.
- `LoadConfig` refuses an empty value, naming `lym config loom --set subagent_model=sonnet` as the repair, because an empty `model:` in a stencil would silently change the subagents' model.
  An existing hub whose `loom.yaml` lacks the key takes the template default only after `lym config reconcile --apply`; until then the load refuses.
- The darn writer receives the value through `wireDarn`'s new `subagentModel` parameter and `darnengine`'s spec, rather than loading `loom.yaml` itself.
- The key is not closed to the local layer.

## To check by hand

- Run `lym config reconcile --apply` on an existing hub and confirm `subagent_model: sonnet` is written, then start a Discussion-Write run and confirm the rendered prompt names `model: "sonnet"`.
- The benchmark of the guidance against a writer given none belongs to `planner-fork-lookups` and was not run here.
