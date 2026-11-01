```yaml
lens: fix-affected-set
outcome: reported
```

Findings: none. The stencil is the only copy of the sweep text, and no other test pins it.

- **F1 (other copies of the second-deviation wording):** only the stencil carries it. Its line 43 and the reworded text agree, and the new text drops the failure-fix label. Line 69 still describes the first deviation correctly.
- **F2 (assertions on the sweep prose):** no test other than the edited case mentions the sweep. The title revert and the two deleted lines cover the whole set.
- **F3 (grep overlap):** the new sentence sits in the stencil's sweep paragraph and refers back to the rename paragraph's zero-reference grep. No other stencil restates that grep.

Unchecked: the review's remark that `git mv` is silent under "after any rename tool run". It is not a finding, so I did not search for it.

Tool calls:
1. `grep -rln` for the sweep sentences across the repo. My context did not show other files containing them.
2. `grep -rn` for "retired name" in `contracts` and `internal`. My context did not show other stencil or test uses of the term.
