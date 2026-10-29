```yaml
lens: fix-affected-set
outcome: reported
```

Findings: none in the F2 or F3 sets. F1 is a `decision`-class finding and was not checked.

- **F2 (CLAUDE.md in the orch stencil set):** `grep -rn "CLAUDE.md" contracts/stencils/orch/` returns no match, so no orch stencil names the file any more.
- **F3 (the "edit rule" phrase):** the only remaining twin is the comment at `contracts/stencils/parentdirective_test.go:141`, which says the stencil "is the only statement of the edit rule". It is a test comment that paraphrases the PATTERN, and the PATTERN is a different file from the stencil header. It is not a finding: the rubric scopes findings to text the fix touched or reaches, and the PATTERN wording is a paraphrase, not a quote, so it stays accurate. The other hits are code comments and research docs that do not state the rule.

Calls made:
1. `grep -rn "CLAUDE.md" contracts/stencils/orch/`: reran F2's search over the orch stencil set, which my context did not hold.
2. `grep -rnE "only statement of the edit|…|edit rule" --include=*.md --include=*.go .`: reran F3's search over the repo for other sites of the phrase, which my context did not hold.
