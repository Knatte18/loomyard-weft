# Skip create-not-done for a Create target under the root `.scratch/`

Fixes #731.

A card could name a git-ignored scratch draft as its `Create:` target and, following the cleanup rule for scratch files, remove that draft once `lyx board upsert` had read it.
`lyx webster record-batch` then resolved the target, got `not_found` and refused the batch with `create-not-done`, although the card had done its work.
The card's removal step and the check could never both hold.

`DoneChecks` in `internal/planglyph/donecheck.go` now skips a `Create` group target whose repository-root-relative resolve key starts with `.scratch/`: it is neither resolved nor judged.
The root `.scratch/` is the directory fabric seeds into each warp repo's `.git/info/exclude`, so nothing under it can be a landed deliverable.

## Design decisions

- The exemption is bounded to `Create` targets under the worktree root's `.scratch/`.
  A `.scratch/` below the root is not git-ignored and keeps its check, as do every `Delete` target and every `Rename` new side.
  Planning-time checks such as `path-missing` are unchanged.
- The prefix lives in an unexported constant in `planglyph`, since the two packages that already name `.scratch` keep it unexported and importing either would pull an engine into `planglyph`.
- A new transient-target marker in the plan format and a planning-time rejection of a card that creates and removes one path were both passed over: the first needs parser and spec changes for a case one directory already identifies, and the second cannot see a removal that lives in card prose.

## Docs and tests

- `internal/planglyph/doc.go` and the `DoneChecks` doc comment state the exemption.
- `contracts/specs/loom-plan-spec.md` tells the planner that a root `.scratch/` `Create` target is a transient draft the card may remove.
- `TestDoneChecks_Verdicts` gains two rows: a removed root `.scratch/` draft yields no finding, and a missing `sub/.scratch/` file still yields `create-not-done`.
