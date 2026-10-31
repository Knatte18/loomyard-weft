# The Bolt checkout directory _board is renamed _bolt

`bolt-dir-rename` · Note · enhancement, fabric, board, config

## Seen (operator, 2026-10-31)

The hub's Bolt checkout is the directory `_board`, but it holds far more than the board: `board.json`, its README and design files, the hub's `_lym` config and orch rules, and soon heddle's database (note `warp-visibility`).
The name says board where the directory is Bolt.

## Idea

Rename the directory `_board` to `_bolt`, the name of the repository it checks out.
- The name sits in `fabricengine.BoardDirName` and `lymcwd`'s own constant, and in many Go files, stencils, skills, contracts and docs that spell the path.
- Existing hubs on Neve and OSL-1046 have `_board` on disk, so the rename needs a rename program as `tapestry-rename` had, run once per hub.
- It touches nearly every package, so it runs alone, with no other run in flight.
- `runproc.HubOwner`, also spelled `_board`, is an owner name, not the directory; whether it follows is to be decided.
