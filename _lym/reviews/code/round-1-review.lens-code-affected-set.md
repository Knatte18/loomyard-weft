```yaml
lens: code-affected-set
outcome: reported
```

**List per change (code-affected-set):**

1. **`git branch` judged by `gitBranchReadsOnly`, in `cluster.go`.**
   - Set found: the single production caller, `gitInvocationMutates`, which feeds `firstMutatingGitCommand`. That function is called at `cluster.go:460` and in the tests at `cluster_test.go:382` and `:439`. `mutatingGitSubcommands` still lists `branch`, so the writing forms still go through. The init, worktree and tag cases, and the help-request and fork-created-repository handling, are untouched. Covered.
2. **`firstMutatingGitCommand` doc comment.** It now names the `git tag` and `git branch` exceptions. Covered.
3. **Package fork-audit paragraph in `doc.go`.** Its `git branch` sentence matches the code's rule: every option read-only, and either no operand or list mode. It also names the `-a`, `-r` and `--format` operand caveat. Covered.
4. **Test header and rows in `cluster_test.go`.** The header comment is updated. The old `git branch -d old` row stays valid. Covered.
5. **Other docs, stencils and PATTERN entries.**
   - Stencils: `burler-template-reviewer.md:24` says only "no mutating git command". Nothing there enumerates `git branch` or `git tag`, so no stale wording.
   - PATTERN: no entry covers the fork audit's git classification, and the diff declares no PATTERN change. Nothing to land.
   - Commit message: it has no `Pattern-Departure:` trailer, and none is needed.
   - Fixtures: none reference this audit.
   - Covered, with no member left wrong.
6. **Separate read-only `git branch` classifier in `internal/fabricengine/refscanner.go:597` (`readOnlyGitBranch`).**
   - It serves a different purpose, ref scanning, and has its own tests at `refscanner_test.go:177` and `:272-278`.
   - Neither the board entry nor the decision record asks for it to change, and the diff does not touch it.
   - Not a finding. It is an adjacent duplicate rule, recorded here only for the handler.

**Findings:** none.

**Tool calls (1 of 6):**
1. `grep` over `*.md`, `*.go`, `*.yaml` and `*.tmpl` for `git branch`, "mutating git" and the `git tag` list wording, outside `cluster.go` and `cluster_test.go`. My context lacked any search for other docs, stencils or classifiers that describe the audit.
