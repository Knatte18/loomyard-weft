# Batten re-adopts a gone pair and orders a slug's remote work

A batten run no longer fails pass after pass when its task worktree is gone, and a slug's remote work now stays in order.
A teardown's archive tag never goes out ahead of the weft tip it archives, and no removal or deletion of a slug's former pair touches a later run's branches.

Closes #9
Closes #15

## What changes

- **Resume re-adopts a gone pair.**
  `lym batten run` resuming a run whose task worktree is gone, at a row other than Worktree-Create and Worktree-Teardown, judges the entry from origin, prunes stale worktree registrations and calls `fabricengine.Topology.Add` under a new adopt-only option.
  Adopt-only forks nothing: it re-adopts a live weft and task branch, reuses a standing records worktree, and returns a not-live sentinel otherwise.
  The pair push is then spawned and the origin reconcile runs again with the pair's branches judged.
  The run-scope origin reconcile gains a resume-only option under which an absent pair worktree is reported as absent instead of an error; every other caller keeps today's error.
- **A pair that cannot be re-adopted fails the run once.**
  Only a closed set of typed refusals from `Add` writes a `failed` stop, posts one notice to the parent and exits non-zero.
  Any other answer refuses the pass as before, so dobby retries.
  The stop names the way forward by row: `lym batten cancel <slug> --remove` before Land; at Land, the restore of each missing branch from the records tag whose commit holds the child's `done` status, then `lym batten run <slug>`.
  The new code is in `internal/battencli/readopt.go` and `internal/battencli/readoptstop.go`.
- **Ways forward that name `lym fabric remove` also settle the record.**
  Such a stop for a slug with a batten run record names the cancel (before Land) or the Land way forward after it.
  `lym fabric remove` itself warns in its result for such a slug and changes no batten record.
  Finalize's stop for a parent that already holds the work is the exception and keeps leaving the owner pause standing.
- **Teardown orders its pushes.**
  `fabricengine.Topology.Remove` pushes the weft tip alone before the archive tag, archives a diverged origin tip as well, and retries an archive-tag push refused with "missing necessary objects".
  A failed branch push is logged and never fails the removal.
- **Weft deletes on origin are leased.**
  A removal, its origin-only path and the cleanup sweep delete the weft branch under a lease on the origin tip they archived.
  A delete the lease refuses keeps the branch and is reported as kept because origin moved.
- **Per-slug teardown lock.**
  `Add` and the task-branch discard take the lock exclusive, the cleanup sweep tries it once per slug and skips a held slug, and every push of a pair's weft branch, `commit-records` included, takes it shared.
- **`Add` drops a former pair's stale remote-tracking ref and upstream config** for a branch origin lacks.
  This was the cause of the 2026-11-03 failure, where a new run's reconcile read its own weft branch as gone before its first push landed.
- **Tackler stencil** gains a rule to read the remote tip with `git ls-remote` before judging a commit or branch absent from a remote.

## Design decisions

- `AdoptLocal` is now also set by the batten resume under the adopt-only option, no longer only by `--adopt-local`; the adopt-only option forks no pair over a stale branch.
- The resume treats a path at the task worktree's location that does not resolve as a worktree as absent, so a stray path reaches `Add`'s typed refusal instead of failing every pass.
- `Remove` now waits on one more network push and holds the teardown lock longer; a hand `lym fabric add` or a cancel's discard that meets the hold is refused before any mutation, and a resume's re-adopt that meets it is retried by dobby.
- A `commit-records` commit made after the teardown read the local tip is given up with the removal and is not covered by an archive tag.
- `PATTERN-slug-remote-order` and `PATTERN-push-both-sides` are updated to match.

## To check by hand

- Prunable-registration handling: a prunable task worktree registration counts as an absent pair, and the reconcile may skip judging that pair.
  Check the case in `internal/fabricengine/originreconcile.go` and the resume path.
- The archive test reads an unexported field; check it in `internal/fabricengine/` tests.
- The cleanup sweep's re-read of a live pair is not reached by any test.
- The follow-up sites of the `lym fabric remove` way forward are enumerated by hand in `internal/pairteardown/followup.go`, `internal/landingshed/stuck.go` and `internal/tendercli/deps.go`; check no site that names `lym fabric remove` was missed.
- The tackler's `git ls-remote` rule is static prose with no test; read the rendered prompt.
- The integration suite and the end-to-end resume of a real run with a deleted task worktree were not run by hand.
