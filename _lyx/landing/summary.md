# Webster and plan validity: refuse backward plans, fix stops

A plan whose card order is not executable no longer reaches Webster, and the Webster, reset and Publish stops reported from the `skills-in-one-turn`, `verify-publish-hardening` and `test-suite-measure` runs either cannot recur or name a way forward that works the first time.
Each fix has a regression test that drives the real CLI wiring.

Closes #397
Closes #401
Closes #402
Closes #403
Closes #404

## Plan-Gate

A valid plan is executable in card-number order, so two new blocking checks refuse the shapes that broke runs.
They run wherever the format and glyph validation runs: the Plan-Gate, the Rework-Plan-Gate, `lyx webster validate`, run entry and `begin-batch`'s dispatch validation.

- `uses-later-target` (`internal/planparser`): a card whose `Uses:` names an entry in the targets of a higher-numbered card.
  The match rule mirrors `deriveEdges` in `internal/websterengine/sequence.go` exactly.
  Once no `Uses:` edge points backward, every derived edge points forward, so a cycle needs no check of its own.
- `delete-before-reference` (`internal/planglyph`): a card deletes a member or unit glyph that a later card's `Edit:` code still references by identifier.
  quarry has no reference API, so the match is a whole-word text match inside the later card's resolved spans.
  It can flag an unrelated symbol of the same name; the fix it names, moving the delete after the later card, is always available.

Both checks have rows in `contracts/specs/loom-plan-spec.md` and in the owning package's `doc.go`.
At dispatch a finding is reported only when its consumer card is unbegun.

## Webster

- A **Delete:** target already gone at dispatch no longer blocks `begin-batch`.
  `ValidateDispatch` downgrades it to an informational `delete-target-gone` advisory, once at least one card is begun.
  `lyx webster validate` and the Plan-Gate still refuse a missing Delete target, and a missing `Edit:`, `Uses:` or Rename-old target still blocks.
- A `delete-not-done` caused by a later card's reference names that card and the plan-edit way forward: move the delete, `lyx webster rebaseline --card NN`, then `recover-batch NN`.
  `recover-batch` runs the same check and refuses with `batch_failed` instead of looping.
  A batch whose record lists uncheckable paths gets the `run --fresh` steps instead.
- `begin-batch` archives a report that has no begin-batch record and returns `archived_report`.
  A recorded batch's report still refuses, and the message now states the record it saw.
- `rebaseline --card NN` accepts an edit to a card of a `failed`, `dead` or `stuck` batch and restamps only that card's hash.
  A `done` or non-terminal batch, and a failed batch with uncheckable paths, still refuse, each with its own reason.
  For a dead batch the restamp stops no strand; `recover-batch` stops it before it spawns.
- The `webster-background-shell` note and the matching `RunResult.Warnings` entry state that the wait stopped after `background_shell_wait_min`, that lyx did not stop the shell, and the run's outcome after the turn end.
  The behavior is unchanged.
- `begin-batch` drops its per-batch model assertion: the injector seam, `State.AssertedModel` with its launch-time write, `BeginResult.AssertedModel`, the sandbox assertions and the "model switch injection failed" refusal row are removed.
  A `state.json` that still carries `assertedModel` loads unchanged.

## Reset and Publish

- `lyx webster reset` now moves the remote task branch to the reset commit, through a lease-guarded update in `internal/fabricengine/destroy.go` that runs through `gitexec` and appends to the reset's mutation record.
  `gitrepo` gains `UpdateRemoteBranchLeased` (with `ErrLeaseRejected`) and `CommitsNotIn`, and `Fabric.RemoteOnlyCommits` reports the remote tip and the commits the local HEAD lacks.
  The order is the local gate checks, a fetch with an ancestry check, the remote update, then the checkout rewrite.
- The ancestry check refuses when the remote holds commits the local HEAD lacks, names them, and refuses before any mutation.
  For the run's own abandoned commits (#404's state) it names `git merge --strategy ours origin/<task-branch>` and a re-run; for commits that may be someone else's it names no merge and says not to re-run.
  The lease is on the tip the check saw, so a tip that moved after the fetch refuses too.
- The update is skipped when the remote tip equals or is an ancestor of the reset commit, the remote has no such branch, no remote is configured, or under `SkipPush`.
  A failure after the remote moved reports `partial` with the mutation record, and a re-run converges.
- Publish keeps escalating on `gitrepo.ErrPushRejected` without retry, since it merges the parent in before pushing and a retry cannot change the outcome.
  The stuck reason now names the remote tip, the count of commits the local branch lacks, and the way forward: merge `origin/<task-branch>`, then resume the run.

Every changed refusal has its `contracts/specs/refusal-spec.md` row.

## Design decisions for review

- Plan-Gate `uses-later-target` refuses plans that pass today and that Webster only runs because `SequenceBatches` reorders them.
  Plan-Write may bounce more until `webster-delete-ordering` updates the plan spec's prose and template.
- Webster is not changed to drop `SequenceBatches`: it stays a no-op on gated plans, and its removal stays with the board note `webster-delete-ordering`.
- The remote update in `webster reset` is a force update, denied to agents, so lyx performs it itself.
  The ancestry check and the lease are the guard, and the operator's reading of the listed commits before running the `ours` merge is the one judgment the guard leaves open: running that merge on someone else's commits discards them on the re-run.
- `delete-before-reference` does not see references from code no later card edits, or from a symbol defined in a deleted file; those still surface at the batch's verify or the review.

## To check by hand

- The plan-level `## verify:` was not run by the session that drove the batches; the verify gate runs it at HEAD.
- `lyx webster reset` against a real hosted remote, including the network and authentication failure paths, was not exercised; the tests build their remotes through `hubforge`.
- This change edits the stencils `loom-rubric-plan-review` and `webster-template-master`, so the board copies stay stale until a production deploy (`update-plugins.sh`) runs.
