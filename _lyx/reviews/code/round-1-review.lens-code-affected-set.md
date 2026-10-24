```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** I looked for everything the diff's changes force outside the lines it touches.

Per change, the set I found:

- **`commandRunsMutatingGit` replaced by `firstMutatingGitCommand`, and `gitSubcommand` removed by renaming it to `parseGitInvocation`.** I found no references to either old name left in the repository. The only callers of the new function are `forkViolation` (`cluster.go:395`) and, through it, `auditClusterRound` and `auditFixRound` (`cluster.go:376`, `cluster.go:424`). Covered.
- **The matrix test `TestCommandRunsMutatingGit` became `TestFirstMutatingGitCommand`.** The header comment of `cluster_test.go` is updated to match. Covered.
- **The violation message is unchanged.** The only assertion on it is the `"git-mutating command"` substring at `cluster_test.go:175`, and the message text at `cluster.go:397` still contains it. Covered.
- **The behaviour change, where a mutating git call inside a fork-created repository no longer counts.**
  - Docs: the `doc.go` fork-discipline paragraph at lines 301–305 is updated.
  - Stencils: this lens found no fork stencil naming the git rule. `grep` over the repository for "state-mutating git" and "git-mutating" matched nothing outside `internal/burlerengine`, so there is no stencil to update, and the record puts the fork stencils out of scope.
  - Other users: `forkaudit.go:32` and `:38` only describe `ParentBashCommands` and defer to the caller to classify git-mutating commands, so they stay correct.

**Findings:** none. No member of the set is left wrong.

Tool calls:
1. `grep -rIn "git-mutating|state-mutating git|forkViolation|mutatingGitSubcommands"` over the repository: to find other callers, docs and stencils, which my context did not hold.
