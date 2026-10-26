```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **Heading row.** The "skipped row of a visited review segment leaves the heading done" row fails with the defect in place. If `checklistReviewBlock` passed `skipped[row.Name]` instead of `false`, `Rev-Fix` (no visit, skipped) would return `[-]`. That is neither done nor not-reached, so the heading loop would take it as a state box and render `[-] 2. Rev`. The row expects `[x]`.
- **Current-producer rows.** Both the running and failed rows fail if `checklistBox` checked `skipped` before the current-producer switch. They would render `[-]`, and the rows expect `[>]` and `[!]`. The failed row passed on the first run and is a distinct `State` branch, so it is not redundant.
- **Review's `[=]` case.** The review offered `[>]` or `[=]`. The fix pins `[>]` and `[!]`, not `[=]`. All three states share one `switch` in `checklistBox`, so a precedence regression would show in either row. This leaves no stale or wrong text and is below the rubric's bar.
- **Placement and expected lines.** The rows sit beside the skipped-segment row and use the existing `recipe` and `reviewRecipe` fixtures, so they break no sibling row. I did not recheck the expected lines; the gate run on `./internal/loomcli` passed.

Further tool calls: none.
