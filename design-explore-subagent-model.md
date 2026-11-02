# Discussion writers hand lookups to Sonnet subagents, and the subagent model is config

`explore-subagent-model` · Task · enhancement, stencil, loom, darn, config · running · Record-Write

## Seen (orch, 2026-11-01)

Every lookup a Plan or Discussion writer makes is paid at its whole context, so exploration late in a large session is the expensive part of planning.
Discussion-Write explores the code more than any other role.
Task `darn-sonnet-helpers` landed the guidance for the Plan writer and the darn writer to hand batches of independent lookups to fresh Sonnet subagents (`contracts/stencils/loom/loom-template-plan.md`, `contracts/stencils/darn/darn-template-write.md`).
The Discussion-Write stencil has no such guidance, and each stencil that has it hardcodes `model: "sonnet"`.

## Design (operator, 2026-11-02)

1. The Discussion-Write stencil gets the same guidance as the Plan stencil, in the same form:
   - the writer hands a batch of independent lookups to fresh subagents, launched together in one message, each prompt carrying its whole question and asking for a short answer naming files and spans;
   - the writer is encouraged, not commanded: it keeps its own lookups when it judges them small or needs to see the code itself;
   - a fork does not serve here, since it inherits the writer's whole context.
2. The subagent model is config, not a literal in the stencils: one new key beside the writer-model keys of the loom config, with `sonnet` as the template default so behaviour is unchanged.
   The Plan, Discussion-Write and darn-write stencils take the model from it, in place of the hardcoded `model: "sonnet"`.
   The writer picks the key's name and its module from the repository's config conventions, and says in the key's template comment which stencils read it.

The bench of the guidance, against a writer given none, is `planner-fork-lookups`'s and is no part of this task.
