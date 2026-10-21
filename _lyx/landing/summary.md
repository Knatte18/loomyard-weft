# webster: skip create-not-done for a root .scratch/ Create target

A card that names a git-ignored scratch draft as its `Create:` target and removes the draft once read no longer makes `lyx webster record-batch` refuse the batch with `create-not-done`.
The draft typically feeds an effect outside the repo, such as a board write, so after the batch it is gone and quarry answers `not_found`.
Before this change, a planner that followed the scratch-file cleanup rule produced a card Webster always rejected.

`planglyph.DoneChecks` now skips a `Create` group target whose resolve key starts with `.scratch/`: it is neither resolved nor reported.
`internal/planglyph/doc.go` and `contracts/specs/loom-plan-spec.md` state the exemption, so the planner learns that such a target is transient and may be removed by its card.

Closes #731

## Design decisions

- The exemption is the issue's first remedy (exempt `.scratch/` paths), not a new transient target type.
  It needs no plan-format change and covers the reported card unchanged.
- Only the worktree root's `.scratch/` is exempt, since that is the only one git ignores, and nothing there can be a landed deliverable.
  Plan refs reach `DoneChecks` already canonicalized to root-relative spellings, so a prefix test on the resolve key names exactly the root directory.
- A `.scratch/` below the root, a `Rename` new side and every `Delete` target keep their checks, and the planning-time checks such as `path-missing` are unchanged.
- The prefix is an unexported constant in `internal/planglyph`, as in the other packages that name `.scratch`; importing one of them would pull in an unrelated engine.
- Not added: a planning-time check rejecting a card that creates and removes one path.

## Risk

A planner could name a `.scratch/` `Create` target for a file it means to keep.
That file is git-ignored and never lands, so the cost is a lost draft, not a false landing.

## Tests

`TestDoneChecks_Verdicts` gains rows for an absent root `.scratch/` target (no finding) and a non-root `sub/.scratch/` target (still `create-not-done`).
Nothing was left to check or run by hand.
