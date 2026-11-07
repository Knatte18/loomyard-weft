# fabric clone: render the prime after its configs, report a failed render

`lym fabric clone` fired the worktree "made" event for the prime before the per-worktree module configs were written and committed.
It also dropped the warning text the event returns, so a failed render of heddle's files into the prime showed only in the log.
An added worktree fires the event last and was unaffected.

Changes:

- `fabriccli.CloneAndWire` now fires `NotifyWorktreeMade` after the module configs are committed, before the hub walk.
  The prime is then set up as an added pair is when its seam fires.
- A failed render never fails the clone.
  Its warning is returned as `CloneResult.SeamWarning` and printed as `seam_warning` in the clone envelope (always present, empty when none).
  The warning names `lym heddle render` in the prime as the way forward.
- When the hub walk fails after a failed render, the failure envelope has no field for the warning, so the walk's error carries it instead.
- `heddleengine.Seam.Made` now returns an error naming the render's warnings when a target's text stayed unwritten and no file was written.
  Before, such a render (for example, exclude lines missing) only logged, so no caller could surface it.
  `heddleengine.Report` gains `Unwritten` to carry this.
  `TookIn` behaviour is unchanged.
- `hubforge.RegisterTeardown` is now exported so the seam tests can tear down hubs they clone themselves.
- Docs in `fabriccli`, `fabricengine` and `hubforge` follow the new order and field.

Design notes:

- The render reads only the Bolt checkout's heddle store and the shipped layer, so it needs nothing the walk writes.
  A test pins that the event fires before the walk's build stamp exists.
- The `Made` error fires only when no file was written, so a partial render stays a logged warning.
- Which of the two differences left the prime unrendered in the operator's report was not confirmed.
  The new test with the real heddle seam shows a clone now leaves `CLAUDE.local.md` rendered in the prime.

Tests: new cases in `worktreeseam_integration_test.go` cover a failing seam on clone, a failing seam followed by a walk failure, a clone rendered by the real heddle seam, and the `seam_warning` envelope field for failing and healthy seams.
`seam_test.go` covers `Made` with no exclude lines.

To check by hand:

- `clone-envelope-seam-warning-untested`, `clone-reorder-unproven-render`, `made-event-position-unpinned`, `render-warnings-only-logged` and `walk-failure-drops-seam-warning` were open review findings, addressed by the tests and code above; confirm each is covered.
- Run `lym fabric clone` on a real hub and confirm the prime has heddle's files.
- Nothing ran the integration suite for this change.
