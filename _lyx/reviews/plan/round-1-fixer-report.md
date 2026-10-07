# Fixer report — plan, round 1

## Changed

- **F1** (`01-weft-lock-dir.md`): removed the `internal/hubgeom/parent_test.go` case and its `Edit` target; geometry step (b) keeps the lock-directory error assertion and the heal.
- **F2** (`04-add-adopts-live-pair.md`): removed the origin-weft-only row from the `adoptremote_integration_test.go` bullet; the converted `add_remoteleftover_integration_test.go` row pins it, and the both-tips CLI row stays.
- **F3** (`06-finalize-config-notice.md`): removed the fake-seam "already-up-to-date merge queues the notice again" clause; the integration step's second Finalize pins the resume re-queue.
- **F4** (`03-reconcile-adopts-origin-weft.md`): the Intent now states that a branch the resolver created stays in place when the adopt fails, is recorded in the `*Mutations` record, is adopted by the next reconcile, and leaves origin untouched.

The overview needed no change.
`approved:` was not touched.

## Deferred

None.
