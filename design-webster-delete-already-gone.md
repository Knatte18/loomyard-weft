# Webster blocks when a Delete target is already gone

`webster-delete-already-gone` · Notes · bug

Seen 2026-10-03 in the `verify-publish-hardening` run, after 12 batches.
Batch 15 (card 15, `webster-verify-gate`) deleted `internal/websterengine/integration_test.go`, even though card 15 does not list it.
Card 17 lists the file under **Delete:**, so batch 16's `begin-batch` refused with `plan_drifted`: the path does not exist on disk and is not a Create target or Rename destination of any card.
The run blocked until the plan was edited by hand and rebaselined.

A Delete target that is already gone is the state the card wants, so it should not block the run.
Options:
- `begin-batch` accepts a missing **Delete:** path when an earlier batch's commit deleted it, and notes it in the batch record.
- The per-card check flags a batch that deletes a file outside its card, at the batch where it happens rather than at a later card.


From GitHub #352 (intake 2026-10-04).
