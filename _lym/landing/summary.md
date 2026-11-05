# Add `lym beam merge-parent` and merge-in and land buttons

Bringing a beam's parent into the beam was done by writing a merge task by hand.
`lym beam merge-parent <slug>` now files that task in one call, at any time and not only at landing.

## What changes

- `lym beam merge-parent <slug>` (`internal/beamcli/mergeparent.go`) files a released darn task in the beam's project.
  Its run merges the beam's parent branch, the `ParentBranch` of the beam pair's origin record, into the task's branch; its landing brings the merge into the beam.
  The beam's dobby starts it as any member.
- The task's writer runs `lym fabric merge-in`, settles conflicts, makes the gate pass and changes nothing else.
- The task's Code-Review perch is on through its `run_settings`: the soft budget is the hub's resolved Code-Review budget, at least one round, and the hard cap stays above it.
- The task slug is the beam's slug plus `-merge-in`, numbered from `-2` when taken, with the beam's slug cut to fit the board's slug length.
  It carries the beam's labels so the board's type-label rule holds.
- The verb runs anywhere in the hub and refuses, each naming a way forward:
  - an entry that is not an open beam project with a pair here (the check `land` uses);
  - a beam whose origin record names no parent branch, pointing to `lym fabric reconcile`;
  - a beam that holds an open merge task, pointing to `lym batten status` and `lym batten cancel --remove`.
- The Code-Review rubric (`loom-rubric-code-review.md`) narrows the review range for such a task to the merge itself.
  It reviews each merge commit through `git show --remerge-diff` and every non-merge commit in the range, never the parent's own commits the merge brings in.
- Reed's default `beam` button group gains `merge-in` (`lym beam merge-parent {slug}`) and `land` (`lym beam land {slug}`), each with a confirm prompt, in both the POSIX and Windows templates.
- `landDeps` gains `Tasks`, `FileTask` and `CodeReviewCaps`; `docs/overview.md` and the `beamcli` package docs describe the verb.

## Design decisions

- pawl neither starts a merge task nor waits for one; its Parent-Align still brings the parent in at landing whether or not a merge task ran.
- The perch reviews the merge alone; checking the project's goal before the beam lands is no part of it.
- The merge task is a plain darn run: its writer makes the merge and settles conflicts, and the Darn-Write verify gate's gate fixer clears gate failures.
  This departs from the entry's wording that the merge fixer settles conflicts.
  The merge fixer belongs to Parent-Align and Finalize, which merge the task's own parent under its lock; pointing them at the beam's parent would need a second parent branch, lock and merge record through landing and the darn recipe, for no gain.
- The board creates a task as a draft, so filing releases it with a second write.
  If the release fails, the error names the stranded draft and `lym board remove <slug>`.
  The writes carry no strand name, so the run-agent task guard does not judge them.
- The Code-Review budget is read from the prime's loom config, the hub's own, which no run profile reaches.

## To check

- Conflicts in a merge task are settled by the darn writer, not the merge fixer (`internal/beamcli/mergeparent.go`, `mergeParentBody`); confirm this departure is acceptable.
- `deps.go` `FileTask` and `CodeReviewCaps` are tested only through fakes and `dobbydeps_integration_test.go`; run `lym beam merge-parent` against a real beam to see the filed task start and its perch run.
- The prime gets no `merge-in` or `land` button: confirm the `prime` group in both reed templates reads as intended.

Not run by hand: a merge task end to end against a real parent with a conflict.
