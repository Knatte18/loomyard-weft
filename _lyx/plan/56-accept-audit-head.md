# Card 56 — accept-audit-head

**Edit:**
- `internal/websterengine/auditledger.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/acceptaudit.go`
- `internal/websterengine/suspect_test.go`
- `internal/websterengine/auditledger_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/acceptaudit_test.go`

**Uses:**
- `internal/websterengine/gitwrap.go`
- `internal/websterengine/suspect.go`
- `internal/webstercli/cli.go`

**Intent:**
Fix the `accept-audit` half of PR #321 round-4 finding A.
`AcceptPendingAudit` compares the worktree against the last batch head and never looks at HEAD.
Master can commit its suspect write through Bash;
restoring the worktree file then passes `accept-audit` while HEAD still carries the commit, and Publish pushes it.
Nothing after the last batch head is legitimate except a clean parent merge-in, which `reconcileReportHead` already accepts.

In `auditledger.go`:

- `AcceptPendingAudit(st *State, geom Geometry, parentBranch ParentBranchFunc) ([]PendingAuditFinding, error)`.
- When the last batch head is known, it first runs `reconcileReportHead(geom.WorktreeRoot, last, "accept-audit: last batch head", parentBranch)`.
  A refusal is returned wrapped in `ErrAuditNotAcceptable`, mutating nothing;
  its own clause already names moving HEAD back to the head with git, then re-running the verb.
- The path check's base becomes the worktree's HEAD once the reconcile accepts it: the last batch head itself, or a clean parent merge on top of it whose content is legitimate.
  The differing clause names that HEAD in its `git checkout <sha> -- <path>`.
- The doc comment states that the evidence covers HEAD as well as the worktree.

`internal/webstercli/acceptaudit.go` passes `c.parentBranch`, as `record-batch` does, and its `Long` says the verb also refuses while HEAD carries a commit past the last batch head other than a clean parent merge.

`doc.go`'s run-exit paragraph states the HEAD rule.

Tests:

- `suspect_test.go` (integration-tagged):
  - `TestAcceptPendingAudit_RefusesCommitPastHead`: a done batch recorded at C1, a suspect file committed at C2 on top, then restored in the worktree to C1's content → `ErrAuditNotAcceptable` naming C1 and `move HEAD back`, the pending list unchanged.
  - `TestAcceptPendingAudit_AcceptsAfterHeadReset`: the same after `git reset --hard C1` → accepted.
  - Existing `AcceptPendingAudit` calls here and in `auditledger_test.go` pass a nil `ParentBranchFunc`.
- `runlevel_test.go`: the Master-`parent-write`-after-the-last-batch scenario's `AcceptPendingAudit` call passes nil.
- `acceptaudit_test.go`: `TestAcceptAuditCmd_RefusesCommitPastHead` commits a change to the pending finding's tracked file on top of the recorded head and restores the file's content in a second commit, then asserts a non-zero exit naming `move HEAD back` and `state.json` byte-identical.

**ImpactSummary:** accept-audit refuses while HEAD carries a commit past the last batch head other than a clean parent merge, and checks suspect paths against that reconciled HEAD; its signature gains the parent-branch resolver.
