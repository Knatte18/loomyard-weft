```yaml
lens: fix-code-correctness
outcome: reported
```

Task: lens fix-code-correctness over the fixer's changed code and tests. I made no further tool calls.

**MEDIUM — the Codex `background-command` copy contradicts a Master stencil sibling.**
- Changed text: `contracts/stencils/engine/codex/background-command.md`, "act on its output and exit status when it returns".
- Evidence: `webster-template-master.md` line 148 sits right after the block at line 147 and says "You read the digest fields, never the exit code, so the rungs below are unchanged." A Codex Master is told to act on the exit status and, one line later, never to read the exit code.
- Suggested fix (not mine to apply): drop "and exit status" from the Codex copy, so it reads "act on its output when it returns".

**LOW — the discussion row cannot catch an advisor's-engine defect.**
- Changed text: the `a codex chair gets the twill directive worded for codex` subtest in `discussiontable_test.go`, built on `discussionTableConfig(0, false)`.
- Evidence: F1 names "an advisor's engine" as a wrong input `DiscussionTable` could pass. With zero advisors there is no advisor engine to confuse it with. With `discussionTableConfig(2, false)`, chair `sol` and the Claude advisors, a defect that passed an advisor's engine would produce Claude wording and fail the row.

**LOW — redundant wording in the Claude copy at stencil line 156.**
- Changed text: "Run … once more {{template "engine-background-command"}}, then act by these rungs."
- Evidence: the Claude copy already ends "…and act on the command's completion notification", so line 156 renders as "…act on the command's completion notification, then act by these rungs." The wording is slightly redundant but not wrong.

**Traced and found correct**
- **`declarationUnits` and `goLiteralTermViolations`:**
  - `ValueSpec.Names[0]` always exists.
  - The import specs skipped hold no terms.
  - Every top-level declaration is covered, struct tags in `TypeSpec` included.
  - The new `sort.Strings` makes the two-file synthetic row deterministic.
- **Synthetic row line numbers:** `allowed.go` `other` is on line 4 and `x.go` `c` is on line 6, as the expected strings say.
- **F1 begin-batch test:** with Master on `codex` and Recovery on the fake default, a `beginDeps` that read Recovery's choice or the default engine would resolve `fake`, carry the Claude wording, and fail the row.
- **F1 plan test:** a lead engine passed as the default would produce `subagent_type: "fork"` and no `spawn_agent`, and fail.
- **`MerriamBase`:** the in-place stencil edits kept the line count, and each engine-block include counts as one line.

Further tool calls: none, because the context held everything this lens needed.
