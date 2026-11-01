# Rename the Bolt checkout directory _board to _bolt, by script alone

`bolt-dir-rename` · Task · enhancement, fabric, board, config, cwd, stencil · running · Worktree-Create

## Seen (operator, 2026-10-31)

The hub's Bolt checkout is the directory `_board`, but it holds far more than the board: `board.json`, its README and design files, the hub's `_lym` config and orch rules, and heddle's database.
The name says board where the directory is Bolt.

## Design (operator, 2026-10-31)

A pure rename of `_board` to `_bolt`, the name of the repository it checks out, done mechanically by script and nothing else.
- Task `warp-visibility` lands first and makes the name live in one constant, `fabricengine.BoltDirName`, still `_board` (operator, 2026-11-01); this task changes that constant's value to `_bolt`, and every spelling of `_board` still left, in docs and anywhere else, becomes `_bolt`.
- Existing hubs on Neve and OSL-1046 have `_board` on disk, so the task ships a rename program as `tapestry-rename` had, run once per hub.
  It also re-points everything that holds an absolute path into `_board`, such as config, stored reed commands and VS Code setup, and refuses to run while any lym process or run is alive.
- No compatibility: lym itself knows `_bolt` only, with no fallback to `_board`, no detection of the old name and no migration case in its code, since no run on the old name meets the new binary (operator, 2026-10-31).
  The one-shot rename program is the only place the old name appears.
- The rename program stays short, with no what-if code (operator, 2026-10-31): it handles the hubs on Neve and OSL-1046 as they are today, stops and says what it found when anything looks other than expected, leaving that to be fixed by hand, and is deleted from the repository once both hubs are migrated.
- No refactor rides along.
- It runs alone, with no other run in flight.
