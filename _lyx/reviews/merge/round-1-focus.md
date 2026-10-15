---
round: 1
focus:
  - "internal/loomcli/arm.go, cli.go, wiring.go: the parent moved landing wiring out of `loomPreRun` into `wireLanding` and added `loomCLI.languageSkills`, while the task added `card-pack` to `resolveRunID`'s exclusion list, `verbUsesLightweightWiring` and `wireLightweight` (`c.env.StencilsDir`). Settle whether `card-pack` on the lightweight path still works when the parent's wiring runs only in the full `wire()`, and whether the exclusion lists and the verb registration stay consistent after the merge."
  - "internal/loomcli/wiring.go and wiring_test.go: both sides edited `wire`, `wireLightweight` and their tests. Settle whether the tests the task added for `card-pack` and the tests the parent added for `languageSkills` still assert what each meant on the merged `loomCLI`, and whether any overlap or uncovered interaction remains (test economy)."
  - "internal/websterengine/doc.go: the parent added `LanguageSkills` to the Webster master's inputs and the task rewrote the fork-prompt kick-start pack layout. Settle whether the merged doc describes one coherent prompt assembly, and whether the task's per-card pack layout in the fork prompt still holds given the parent's language-skill loading."
  - "contracts/specs/refusal-spec.md and docs/overview.md: both sides edited the same rows. Settle whether the merged text keeps both sides' additions and still matches the code (the `card-pack` verb and the `languageconfig` refusal)."
  - "pattern/PATTERN-config-strictness.md: the parent listed `languageconfig` among the degrading callers. Settle whether the task's code, including the lightweight `card-pack` path, loads no config of its own that breaks the adopt-exactly-one rule."
---

Seed focus from the overlap list.
The parent's side is the `languageSkills` wiring and the `wireLanding` extraction; the task's side is the `card-pack` verb and the Webster fork-prompt change.
