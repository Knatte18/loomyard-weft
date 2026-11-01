```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1:** The added sentence matches `NewCleanGate`'s doc and behaviour: it fails on uncommitted or untracked paths, ignored files excluded, and requires the absolute `Env.WorktreeRoot`. The vocabulary now also lists `parts`, which the closed-vocabulary error message already names.
- **F2:** Both rows, `""` and `"relative/worktree"`, are refused by `requireAbsRoot` before `NewCleanGate` is reached. `NewCleanGate` does no root validation of its own, so deleting the check would make both rows fail the test.
- **F3:** The tapestry recipe has exactly one `Code-Burler-Fix` row, so the name filter cannot match a second row. The gate list is read with the same `[]any` / `map[string]any` shape the existing assertions use. A missing `clean` entry, a reordering or a missing row would each fail the test.

Calls made:
1. `grep` of `NewCleanGate` in `internal/loomshed` and a count of `name: Code-Burler-Fix` in the tapestry recipe. My context lacked `NewCleanGate`'s behaviour and whether the row name is unique.
