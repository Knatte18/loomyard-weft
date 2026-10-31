# The Bolt checkout directory is named in one place only, and renamed _bolt

`bolt-dir-rename` · Note · enhancement, fabric, board, config

## Seen (operator, 2026-10-31)

The hub's Bolt checkout is the directory `_board`, but it holds far more than the board: `board.json`, its README and design files, the hub's `_lym` config and orch rules, and soon heddle's database (note `warp-visibility`).
The name says board where the directory is Bolt.

The name is also spelled out in too many places (operator, 2026-10-31).
The path itself comes from one constant, `fabricengine.BoardDirName`, but:
- `lymcwd` declares its own copy of the constant;
- user-facing messages in `configcli` and `fabricengine` spell `_board` in their text;
- many comments and test files spell it, the tests in place of the constant;
- stencils (landing, burler, drift), the `ly:board` skill and docs spell `_board/...` paths to agents and readers.

## Idea

- Two names for two things (operator, 2026-10-31): `fabricengine.BoltDirName` says where Bolt is checked out, and `fabricengine.BoardDirName` says where the board is.
  Today the board sits at Bolt's root, so `BoardDirName` is set from `BoltDirName`; what lives in Bolt beside the board, such as heddle's database, takes its place from `BoltDirName`, never from `BoardDirName`.
- The directory name lives in that one constant only: `lymcwd` and every message and test take it from there, and a comment names "the Bolt checkout" rather than the path.
- Stencils and skills name the path through what lym already resolves for them, or as "the Bolt checkout", not as a literal.
- Then rename the directory to `_bolt`, the name of the repository it checks out.
- Existing hubs on Neve and OSL-1046 have `_board` on disk, so the rename needs a rename program as `tapestry-rename` had, run once per hub.
- It touches many packages, so it runs alone, with no other run in flight.
- `runproc.HubOwner`, also spelled `_board`, is an owner name, not the directory; it is set from `BoltDirName`, so it follows the rename (operator, 2026-10-31).
