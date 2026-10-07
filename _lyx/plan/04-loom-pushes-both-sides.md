# Card 4 — Loom's status seam and commit-records push the task branch too

**Edit:**
- `internal/loomcli/wiring.go`
- `internal/loomcli/commitrecords.go`
- `internal/loomcli/commitrecords_test.go`
- `internal/loomcli/wiring_commitstatus_integration_test.go`
- `internal/statuscommit/statuscommit.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `PATTERN.md`

**Uses:**
- `plan:internal/fabricengine#PushPairAnchored`
- `plan:internal/fabricengine#IsPushRejected`
- `plan:internal/fabricengine#ErrPushLockBusy`
- `plan:internal/fabricengine#StatusPushLockWait`
- `internal/loomcli#loomCommitStatusDeps`
- `internal/loomcli#commitRecordsVerb`
- `internal/statuscommit#New`

**Intent:**
Decisions 2 and 6 of the decision record.

`loomCommitStatusDeps`'s `Push` calls `fabricengine.PushPairAnchored(location, fabricengine.EnvSyncOptions(), fabricengine.StatusPushLockWait)` instead of `PushAnchored`, so every loom transition and every `lyx loom commit-records` pushes the task branch with the run records, and an agent's raw `git commit` reaches the remote at the next transition.
The statuscommit core is unchanged in behavior: every push error, rejection and lock expiry included, warns and the run continues.
Its doc comments (package doc, `Deps.Push`, the push-warns disposition) and `loomCommitStatusDeps`'s and `commitStatusDeps.Push`'s comments now say the push carries the run records and, in a task pair, the task branch, and that the warning carries the entry's side-named error.
None of these comments or texts in `loomcli` or `statuscommit` names a side (`PATTERN-fabric-vocabulary`).

`commitRecordsVerb`'s push failure splits into two messages, both starting `loom: commit-records: the commit landed locally but the push failed: <err>`, so the envelope no longer claims the records were not pushed; only the appended entry error names a side.
`fabricengine.IsPushRejected(err)` picks the row; a rejection beside another failure takes the rejection row.

- Rejection: `; way forward: merge the remote branch into the local branch in the worktree the error names, or clear what a remote rule objects to, then run lyx fabric push or re-run lyx loom commit-records`.
- Any other failure, a lock expiry included: `; way forward: transient, run lyx fabric push or re-run lyx loom commit-records`.

Neither names a command the agents' settings deny.
`contracts/specs/refusal-spec.md`'s `commit-records: not pushed` row splits into `commit-records: push rejected` and `commit-records: push failed`, both of class transient, with these ways forward.
`docs/overview.md`'s `commit-records` paragraph says it pushes the task branch along with the run records.

`PATTERN.md` gains, under "Fabric and git":
`PATTERN-push-both-sides` — Pushing a run's records side from Go: in a task pair the code branch goes with it, rebase-free, under fabric's absorbing push lock, and pushing is never left to an agent; the prime's per-transition push stays records-only, since its code branch is the operator's parent branch.

Tests:

- `commitrecords_test.go`'s `TestCommitRecordsVerb_ErrorEnvelopes` replaces its `"not pushed"` row with rows reaching each message: a stub error wrapping `gitrepo.ErrPushRejected` reaches the rejection way forward; a plain error and an error wrapping `fabricengine.ErrPushLockBusy` each reach the transient one; an `errors.Join` of a rejected and a plain error reaches the rejection way forward and carries both texts.
  Each row asserts the envelope does not say `not pushed`, and the existing re-run assertion stays.
- `wiring_commitstatus_integration_test.go`'s `TestCommitStatusSeam_Real` gains steps on its real pair:
  after a raw `git commit` in the task worktree, the seam leaves the remote task branch at the local HEAD along with the records branch;
  before the existing remote-failure steps, with the task branch diverged on its remote by a second clone, `commitRecordsVerb` returns the rejection envelope naming the task worktree's path, the records remote is at the records HEAD, and the task branch's local HEAD is unchanged.
  The existing rejected-push and unreachable-remote steps keep asserting the seam warns and returns nil.

**ImpactSummary:** one production caller (`loomCommitStatusDeps`'s `Push`) moves to `PushPairAnchored`; one error message and one spec row split in two, and the one test asserting the old text is updated in this card; `statuscommit` changes in comments only, and `battencli` is untouched.

**Commit:** 4: loom: push the task branch with the run records
