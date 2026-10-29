```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the F2 and F3 changes against their siblings and twins.

- **F2** (`contracts/stencils/orch/orch-template-role.md:188`): the orch stencils contain no `CLAUDE.md` mention, which matches D4. The new wording "lym adds no instruction file to a repository" collides with no other stencil or PATTERN sentence. The other "instruction file" hits (`burler-template-reviewer.md`, `PATTERN-stencil-ownership.md`) use the term in a different sense.
- **F3** (`pattern/PATTERN-parent-directive.md:13`): the new wording matches `contracts/stencils/edit/edit-directive.md:2`, "It is its rules' only statement".
- **Stale twin, out of the review's scope:** the `//testtiming:keep` comment at `contracts/stencils/parentdirective_test.go:141` still says "the edit-directive stencil is the only statement of the edit rule". That is the old PATTERN wording. It is a comment on a test, so it does not break anything. Whoever owns that file can align it if they want it to match.

Tool calls:
1. `grep` for "only statement", "CLAUDE.md" in the orch stencils and "instruction file". My context held neither the twins of the changed sentences nor the rest of the stencils.
