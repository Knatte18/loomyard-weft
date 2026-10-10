# Card 14 — docs-name-the-recipe

**Edit:**
- `internal/boardcli#storeVerbs`

**Prosa:**
- `contracts/stencils/orch/orch-template-role.md`
- `plugins/ly/skills/board/SKILL.md`
- `docs/overview.md`

**Intent:** D12: the operator-facing surfaces name the recipe.
The board CLI's upsert help describes `recipe` as `loom` or `darn`, empty meaning `loom`;
the `ly:board` skill's recipe sentence names the two values.
The orch role stencil's run loop says the batten run drives the task's run, loom or darn, and that a darn run has no parent review and no circling, PR-Gate being its one review point, where a reject re-spawns the darn writer with the findings;
its PR-Gate section says the same reject sends a darn run's findings to a fresh `Darn` spawn.
The overview's module table rows for `internal/loomrecipe` and `internal/shedcli` name both recipes, the batten paragraph names the darn child, and the execution-stack description of the nested batten shed names the task's run as loom or darn.

**ImpactSummary:** Help text and prose only; no code path changes.
