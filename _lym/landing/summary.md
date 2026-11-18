# landingshed: one ending type and merged helpers, no behaviour change

Pilot for `design-simplify-callers`: simplifies the design spots the shrink study named in `internal/landingshed`,
to measure whether its line estimates hold before the same pass is aimed at `fabricengine`.
No behaviour changes; verdicts, reasons and errors are as before, apart from the one message wording noted below.

## What changes

- **One ending type.**
  Every landing step returned `(Outcome, OutputPointer, error, bool)`, or in PR-Review `(Outcome, OutputPointer, bool, error)`,
  and each caller unpacked and repacked the four values.
  A step now returns `*ending`, nil to go on, built by `fail`, `stuck`, `halt` and `gateEnding`.
  The per-producer stop wrappers (`errorStop`, `stuckOrCancelled`, `gateStop`, `alignError`, `haltStop`, `ownerHalt`,
  `landHalt`, `prStopped` and the local closures) are gone.
- **Merged near-identical helpers.**
  - Finalize runs its merge, gate and parent-side merge in one loop that retries once after a merge-in-required error,
    and reads whether the parent holds the task's work once per pass for the owner checks, the land check and the sync;
    it reads it again after the owner checks, because their reconcile can move the task branch.
  - Finalize and PR-Review share the GitHub-failure judgement, the task push, the pull request status read, close,
    find-or-open, record commit and removal, and the landing text read.
  - Finalize and Parent-Align share `newResolver`, the landing gate with its failure recorder, the read before the merge
    and `readRowAttempts`.
  - The fixer and fix review report parsers share `parseVerdictReport`.
  - The fixer, fix review and Describe tables share one `soloSession` builder (`shuttleparity/seats.go` follows its holder glyph constant).
  - The fixer and fix review runs share `startFailure`, and the landing and pr-review gates one command read and marked verify.
- **Seams removed.**
  The `parentPuller` interface had no user.
  PR-Review's wiring check reads every `PullRequestDeps` field by reflection and names the first unset one the row needs,
  in place of a case per field; the `PullRequestDeps` doc names the four fields the row runs without.
  Finalize's hand-over returns its halt directly instead of parking the error in a struct field,
  the merge into the task reads the resolver's result in place of a one-field adapter type,
  and the flaky-pass callback and the credit block read are inlined at their one caller.

One wording change: a missing change description at Finalize now reads "read the change description", as at PR-Review,
in place of "parse change description".

## Left as is

The lock-freed, requeue and rebase layer around the fixer (`freeLock`, `lockFreed`, `rebaseAfterParentMerge` in `finalize.go`) stays.
Freeing the parent lock lets other runs land while the fixer works, and the run then re-merges the parent anew at the front of the queue;
running the fixer outside the locked pass would need that same re-merge and requeue, so the layer would only move.
Reviewer: check that this reading holds against the spec's invariants.

## Lines

Production code in `internal/landingshed` and `internal/shuttleparity`, from `git diff --numstat` against the merge base,
comment lines included: 1037 added, 1315 removed, 278 fewer.
By spot, from the commits that made each:

| Spot | Added | Removed | Net |
| --- | --- | --- | --- |
| One ending type | 473 | 607 | -134 |
| Merged helpers | 566 | 638 | -72 |
| Seams removed | 57 | 154 | -97 |

The three rows sum to 303 fewer; the later fixes, which add the re-read of parent-holds and its doc lines, bring the total to 278.
The study estimated about 600 to 800 lines saved by these spots, so the pilot saved well under half of the low end.
The lock-freed layer, the largest remaining spot, was left, and is part of the gap.

Test lines changed: 17 added and 1 removed across `prreview_test.go` and `prreview_wait_test.go`.
The additions pin each optional PR-Review seam with a row that leaves it unset.
`prreview_wait_test.go` changes one line for a changed signature.

## To check

- The existing tests pass, changed only where a signature they call changes; no full run was recorded here, so run `lym gate test ./internal/landingshed`.
- The lock-freed layer's reason above against the spec's invariants.
