# Round 1 fixer-report

## Changes

- **F1 — fixed as proposed.**
  A tier-1 subtest in `worktreeseam_integration_test.go` installs the real `heddleengine.Seam`, clones with `fabriccli.CloneAndWire` and asserts the prime's `CLAUDE.local.md` carries the heddle marker.
  It passes with the made event where it is, so the event was not moved.
  The comments at `clone.go` and the sequence claim in `fabricengine/doc.go` now say the render reads only the Bolt store and the shipped layer.
- **F2 — fixed as proposed.**
  `CloneAndWire` wraps a `hubreconcile.Ensure` error with the seam warning when one is set, so `errWithRecord` prints it.
  The clone stays intact.
  `CloneAndWire`'s doc, `fabricengine/doc.go`, `fabriccli/doc.go` and the `CloneResult.SeamWarning` field doc name the walk-failure path.
- **F3 — fixed differently inside the proposal.**
  `Seam.Made` returns an error naming the render's warnings when no file was written.
  The first version tested `len(Warnings) > 0`.
  The fix-proposal lens showed that an unrelated warning, such as an override warning on a store with no entries, would then raise a false error.
  `Report` now carries `Unwritten`, the files of targets that held text and were skipped with a warning, and `Made` errors only when `Unwritten` is non-empty and nothing was written.
  `TookIn` is unchanged.
  `seam_test.go` has a `MadeNamesTheWarningsWhenItWritesNothing` row.
- **F4 — fixed in part.**
  The fake seam records whether the hub walk had run when the made event fired, via the build stamp, and the clone subtest asserts it had not.
  A second subtest forces a walk failure by blocking the stamp path from an `onMade` hook and asserts the made event was heard before the clone failed.
  The proposed check that the module-config commit is in the weft history cannot be pinned: with this fixture the commit never happens (a first attempt failed on exactly that), and a clean-status check passes at any position.
  So the event's position against the module-config commit stays unpinned.
- **F5 — fixed as proposed.**
  A table subtest runs `lym fabric clone` through `RunCLIIn` with a failing and a healthy seam.
  It asserts `seam_warning` names `lym heddle render` and the prime for the first, and is the empty string for the second.

Commits: `f4017523`, `53c0061a` and the follow-up on this branch.
F1, F2, F4 and F5 share a commit, because they edit one test file and `clone.go` together.

## Disputed

None.

## Deferred

- The residual F4 gap above: the made event's position against the module-config commit.
  Pinning it needs a fixture whose clone writes a per-worktree config, which is a new `hubforge` fixture and outside this round's files.
- `fabricengine/add.go` calls `NotifyWorktreeMade` for an added pair and drops the returned warning, so a warn-only render there reaches only the log.
  The review's findings name the clone path only, and a pair-add result has no warning field to carry it; that is a design call for the operator.

## Gate repair

The gate refused two comments with a wrapped break: the `CloneAndWire` doc in `fabriccli/clone.go` and the `Seam.Made` doc in `heddleengine/seam.go`.
Each is now one sentence per line, with no wording change.

## Self-check

- fix-consistency, LOW: `fabriccli/doc.go` clone doc stale on the walk-failure path — fixed as proposed.
- fix-consistency, LOW: `CloneResult.SeamWarning` field doc silent on the walk-failure path — fixed as proposed.
- fix-code-correctness, MEDIUM: the `configsCommitted` status check cannot fail on this fixture — fixed with my own change: the vacuous assertion was removed, and the residual gap is deferred above.
- fix-code-correctness, LOW: the `onMade` hook runs under the fake's mutex — not changed; no hook calls back into the seam, and the field comment states the hook runs after the event is recorded.
- fix-proposal, LOW: the F3 count-based condition errors on an unrelated warning — fixed with my own change (`Report.Unwritten`).
- fix-proposal, MEDIUM: the same `configsCommitted` finding as fix-code-correctness — merged, same disposition.
- fix-affected-set, LOW: the pair-add caller drops the made-event warning — deferred, as above.

No fork returned without a report.
