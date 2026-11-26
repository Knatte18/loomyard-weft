```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check of the F1 to F3 fix change set.

Findings left unchecked: none.

no findings

- **F3 (stale names for the removed focus-finding permission):** the repo-wide rerun finds no remaining test name, field or message of that kind. The one hit, `contracts/stencils/harden/harden-rubric-review.md:58` ("A held focus finding"), is current behaviour, because line 54 still has a hardener round raise `origin: focus` findings.
- **F2 (descriptions of the decision rule):** `internal/shedadapters/doc.go`, `bouncerprompt.go`, the `FirstRaisedGates` field comment in `bouncer.go` and line 46 of the judge stencil all agree with the edited comment.
- **F1 (wiring coverage):** the premise concerns the single `bouncerCfg.FirstRaisedGates = true` line. The new table covers it both with and without `harden_round`.

Tool calls:
1. Repo-wide grep for focus-finding wording: my context only held a search limited to `internal`, `contracts` and `docs` with narrower patterns.
2. Grep for decision-rule and first-raised descriptions in non-test files: my context did not show every site that describes the rule.
3. Grep of `harden-rubric-review.md` for `origin` and `held`: my context showed only line 58, not whether the focus finding it names still exists.
