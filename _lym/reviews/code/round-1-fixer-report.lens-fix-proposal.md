```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F2, F3, F5, F6, F7, F8, F9, F10), and the reviewer-proposal parts of F1 and F4.

**LOW** — `ghMutationScanPattern` (`internal/shellguard/rows.go`, the F4 change)
- Summary: the quote-aware scan can stop early on an unbalanced apostrophe, so a mutation that the old flat scan denied now passes.
- Evidence: the units are `[^;&|\\'"]`, `\\[^"n]`, `'[^']*'` and the double-quote unit. A bare `'` matches only as the opener of `'[^']*'`.
- Case: `gh api graphql -f query=don't -f q2=mutation` has an apostrophe with no closing quote on the line. No unit matches it, the scan stops, and the command passes. The old `([^;&|\\]|\\[^n])*mutation` denied it.
- Impact: this is a guardrail-only gap, and the unbalanced input is not valid shell. The row's guardrail comment does not name it. If left open on purpose, add it beside the other passes.

No other findings. The challenged proposals hold:
- F3: the cluster pattern `-[A-Za-z]*[ar][A-Za-z]*` does not match `--comment`, `--body-file` or `-b`. Its one false deny is an attached `-b` body containing a or r, which the row comment now names. `-B` with no word end matches only a word-initial `-B`.
- F7: the stencil names only `gh pr view`, `list`, `diff`, `checks`, `gh run view` and `gh api` reads. The earlier repo grep found no other consumer of the removed commands.
- F9: `editWarned` is cleared only by a successful edit, and the persistent-failure row would log two Warns under the old shared flag.
- F5: the closed, merged and zero-baseline rows each fail if their gate or branch is dropped. The unobservable ReviewEnded gate is the fixer's call to disclose.

Further tool calls: none. My context already held the changed files.
