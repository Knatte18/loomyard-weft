---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/loomcli/prfindings_integration_test.go#blockingFindingsRun
    summary: The findings-file path is built twice from the same literals, once where the helper writes it and once where the subtest reads it, a repeated path literal PATTERN-code-conventions forbids.
    fix: "Make blockingFindingsRun return the findings path beside the worktree, as (worktree, findings string), so its filepath.Join(worktree, \".scratch\", \"pr-findings.md\") at line 101 is the only one. In TestPRFindingsCLI_SendsBackBlockingFindings, take both values with worktree, findings := blockingFindingsRun(t, tt.recipe) and delete the second filepath.Join at line 64. Update the helper's doc comment to say it returns the code worktree and the findings file's path."
    origin: assist:assist
---

### [BLOCKING] The findings-file path is built in two places

**Location:** internal/loomcli/prfindings_integration_test.go#blockingFindingsRun

**Issue:** At the base, the test built `filepath.Join(worktree, ".scratch", "pr-findings.md")` once.
The diff moves the writer into `blockingFindingsRun` (line 101) and has the subtest rebuild the same path from the same literals (line 64).
The verb reads the file the subtest names, and the helper writes the file it names, so the two must agree.
An edit to either literal alone would make the verb read a file that does not exist.
`PATTERN-code-conventions` ("Named constants") says a path that can go stale is declared once and referenced everywhere, never repeated as a literal.
The constraints section makes every violation of a listed entry BLOCKING, however small.
The helper already knows the path, so returning it is enough; no new constant is needed.

The rest of the range matches the amended entry.
The parent's two added entries ask for a darn-path test in `internal/loomcli` that drives `pr-findings` through the real cobra tree, and for the entry to land as overtaken by 6fa7f1740.
The table's Darn row does this.
`contracts/recipes/darn-recipe.yaml` gives darn a PR-Review row rerouting to Code-Bouncer and a Finalize landing row, so the row reaches `wireLanding` and `newShed` in `prReviewSendBackEdge`.
The change description says the entry is overtaken.
6fa7f1740 is an ancestor of the merge base, and its message matches the summary's account.
The module declares Go 1.26, so the parallel subtests capture `tt` safely.

## Rejected

- No `Root-Cause:` trailer on the range: the entry as the parent amended it lands as overtaken, with no fix in the diff, so there is no defect for a trailer to name.
- The Tapestry row no longer exercises the seedless default recipe: the recipe fallback is pinned by other `internal/loomcli` tests, and this test pins the landing wiring, which the explicit seed does not change.
