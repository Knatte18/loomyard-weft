# Card 55 — evidence-base-by-ancestry

**Edit:**
- `internal/websterengine/suspect.go`
- `internal/websterengine/auditledger.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/rebaseline.go`
- `internal/webstercli/rebaseline.go`
- `internal/websterengine/suspect_test.go`
- `internal/websterengine/rebaseline_test.go`

**Uses:**
- `internal/gitrepo/ancestry.go`
- `internal/gitrepo/gitrepo.go`
- `internal/websterengine/sequence.go`
- `internal/websterengine/gitwrap.go`

**Intent:**
Fix PR #321 round-4 finding E and the unresolvable-base item of finding G.
`lastBatchHead` takes the highest-numbered terminal batch and `runStartCommit` the lowest-numbered batch's `StartSHA`, but `SequenceBatches` can run batch 02 before 01.
The named `git checkout <head> -- <path>` can then revert a later batch's committed work, and `--fresh` checks against a mid-run commit.
Separately, a recorded SHA that is not in the repository (a cross-machine resume whose last batch commit was never pushed) makes `worktreePathDiffers` return a raw `exit 128` from `accept-audit` and `--fresh`.

In `suspect.go`, replace `lastBatchHead` and `runStartCommit` with one function that picks both by git ancestry:

- `type evidenceBases struct` with `Start`, `Last` and `Missing []string`.
- `runEvidenceBases(worktree string, st *State) (evidenceBases, error)`, the integration key excluded throughout:
  - `Missing` is every recorded `StartSHA` and terminal `Digest.HeadSHA` that `gitrepo.Repo.SHAExists` does not find, sorted and deduplicated.
  - `Start` is the recorded `StartSHA` that is an ancestor of every other recorded `StartSHA` (`IsAncestor`); "" when none is recorded, when none qualifies, or when any recorded `StartSHA` is missing.
  - `Last` is the terminal `Digest.HeadSHA` that every other terminal head is an ancestor of; "" when none is recorded, when none qualifies, or when any terminal head is missing.
    A missing commit blanks the pick rather than being skipped, because picking among the rest would name an older commit and its way forward would revert landed work.
  - The error is an `IsAncestor` failure.
  - A state recording no SHA returns the zero value without running git.
- `missingCommitsClause(missing []string) string` returns `commit(s) <shas> recorded by this run are not in this repository; way forward: fetch the task branch from the machine that ran those batches with git, then re-run the verb`, or "" for an empty list.
- `checkSuspectPaths` checks once that a non-empty `base` resolves (`SHAExists`); an unresolvable base is treated as an empty one, so every tracked path is unverifiable instead of a git exit-128 error.

Callers:

- `AcceptPendingAudit` takes its base from `runEvidenceBases(geom.WorktreeRoot, st).Last` and its start from `.Start`.
  When `Missing` is non-empty it refuses with `ErrAuditNotAcceptable` and `missingCommitsClause` before any path check, mutating nothing.
- `freshPendingDrop` takes its base from `.Start`, keeping the worktree-HEAD fallback only when no batch recorded a start and none is missing.
  When `Missing` is non-empty it refuses with `ErrPendingAuditFindings` and `missingCommitsClause`.
- `RebaselineDeps` gains `Geom Geometry`, which `internal/webstercli/rebaseline.go` fills from `c.geom`.
  `rebaseline.go` names the start commit from `runEvidenceBases(deps.Geom.WorktreeRoot, deps.State).Start` in place of the first `StartSHA` in batch-number order, computed only on the refusal path that names it, and returns an ancestry error with `way forward: transient, re-run "lyx webster rebaseline"`.
- Update each changed function's doc comment to say the pick is by ancestry.

Tests:

- `suspect_test.go` (integration-tagged, over `newSuspectFixture`): `TestLastBatchHeadAndRunStartCommit` becomes `TestRunEvidenceBases_PicksByAncestry` over a state where batch 02 ran first: batch 02's `StartSHA` is C0 and head C1, batch 01's `StartSHA` is C1 and head C2.
  `Start` is C0 and `Last` is C2, which the batch-number pick gets backwards.
  A state recording a head that is not in the repository → `Missing` names it and `Last` is "".
- `TestCheckSuspectPaths` gains a row: a base SHA that is not in the repository → the tracked path is unverifiable and the error is nil.
- `TestAcceptPendingAudit_RefusesMissingCommit`: a state whose last head is not in the repository → `ErrAuditNotAcceptable` naming the SHA and `git fetch`, the pending list unchanged.
- `TestAcceptPendingAudit_UsesExecutionOrderHead`: the reversed-order state above with a pending finding naming a file batch 01 changed, restored to C2 → accepted.
- `rebaseline_test.go`: the refusal that names the run's start commit names C0 on the reversed-order state.

**ImpactSummary:** accept-audit, run --fresh and rebaseline pick the last batch head and the run's start commit by git ancestry instead of batch number, and a recorded commit missing from the repository refuses with a fetch way forward instead of a raw git error.
