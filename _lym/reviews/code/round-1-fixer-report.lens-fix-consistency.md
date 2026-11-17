```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for consistency with its surroundings (lens fix-consistency).

No findings.

- The F1 wording in `internal/lenses/template.yaml` matches the rubric's "A decision record alone" bullet in `contracts/stencils/loom/loom-rubric-code-review.md:21-23`. That bullet takes the board entry under `## Goal` as the measuring stick, amended by the `Added after Discussion` entries.
- `_lym/landing/summary.md` now describes the same plan versus record-alone split as the lens. Its Tests section no longer mentions the removed phrase assertion.
- `docs/dotnet-hub.md` now lists `code-scope` after `code-tests`. That matches the template's `review-code` fan (`template.yaml:425`) and the csharp layer (`sets/csharp.yaml:53`), so the sentence "repeats the template's lenses and appends the new one" is true again.
- `internal/lenses/config_test.go` still expects the four-lens `review-code` order at `:392` and `code-scope` in the `remove:` list at `:713`. Only the `code-scope` phrase row is gone.
- The only live files that name `code-affected-set` are the template, the csharp layer, the test and the dotnet-hub doc, so there is no other twin. The remaining hits are historical research data under `docs/research`, which are measurement records and out of scope.

Calls made:
1. `grep -rn "code-scope"`: to find every live reference to the lens that the change set might leave out of step.
2. `grep` of the rubric's "A decision record alone" bullet: to check the new lens wording against the rubric, which my context did not hold.
3. `grep -rln "code-affected-set"`: to find sibling docs or fans that list the `review-code` members.
