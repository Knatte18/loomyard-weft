# Round 2 fixer report

Every finding in `round-2-review.md` is fixed, in the plan files only.
`approved:` in `00-overview.md` is untouched.
`lyx webster validate` reports only `plan-unapproved` (blocking, expected, as the flag is never set in this round) and informational `create-new-unit` notes.

## Changed

- **F1** (`04-add-adopts-live-pair.md`): the `fabriccli` add row's last assertion now says the add's push is a fast-forward: origin's warp tip unchanged, origin's weft tip one origin-record commit ahead of the pushed tip, the scratch weft carrying no origin record.
- **F2** (`03-reconcile-adopts-origin-weft.md`): when `adoptWeftWorktree` fails after the resolver created the branch from origin, Reconcile now deletes that branch through `deleteBranch`, as `rollbackSwitch` does.
  A branch that existed locally before is never deleted, and a refused deletion is a logged warning that keeps the adopt error.
  The Intent states why: add's live-pair rule reads a surviving local weft branch as proof of a live pair.
  `deleteBranch` joins the Uses list, and the test list gains a row with a regular file planted at the weft worktree path.
- **F3** (`02-weft-branch-resolver-checkout.md`): `internal/fabricengine/doc.go` joins the Edit list, with a Docs paragraph on checkout's local, then origin, then fork resolution and its unreachable-origin stop.
- **F4** (`04-add-adopts-live-pair.md`): `internal/fabriccli/fabric.go` joins the Edit list, and Docs gains one sentence for the `add` `Long` help text that an existing weft branch is adopted rather than forked.
- **F5** (`06-finalize-config-notice.md`): the "nil seams change nothing" row is replaced by "a non-empty diff with a nil `Notify` still reaches Done", with a note that the other `finalize_test.go` tests already run with both seams nil.

## Deferred

None.
