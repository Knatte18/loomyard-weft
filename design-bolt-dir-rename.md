# Rename the Bolt checkout directory _board to _bolt, by script alone

`bolt-dir-rename` · Task · enhancement, fabric, board, config, cwd, stencil · draft

Depends on: [`run-verbs-shared`](design-run-verbs-shared.md), [`commit-signature-credit`](design-commit-signature-credit.md), [`pawl-loom-recipe`](design-pawl-loom-recipe.md), [`interactive-recipe`](design-interactive-recipe.md)

## Seen (operator, 2026-10-31)

The hub's Bolt checkout is the directory `_board`, but it holds far more than the board: `board.json`, its README and design files, the hub's `_lym` config and orch rules, and soon heddle's database.
The name says board where the directory is Bolt.

## Design (operator, 2026-10-31)

A pure rename of `_board` to `_bolt`, the name of the repository it checks out, done mechanically by script and nothing else.
- Every spelling of `_board` becomes `_bolt`: the constants (`fabricengine.BoardDirName`, the copy in `lymcwd`, `runproc.HubOwner`), user-facing messages, comments, tests, stencils, the `ly:board` skill and docs.
- Existing hubs on Neve and OSL-1046 have `_board` on disk, so the task ships a rename program as `tapestry-rename` had, run once per hub.
- No refactor rides along: splitting a `BoltDirName` from `BoardDirName` and routing every spelling through one constant belong to another task.
- It runs alone, with no other run in flight.
