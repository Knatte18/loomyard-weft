# Rename the Bolt checkout directory from _board to _bolt

The hub's Bolt checkout holds far more than the board: `board.json`, its README and design files, the hub's `_lym` config and orch rules, and heddle's database.
The directory is named after the repository it checks out, so `_board` becomes `_bolt`.

## What changes

- `lymcwd.BoltDirName` changes from `_board` to `_bolt`.
  Every other reader takes the name from that constant.
- Every remaining spelling of `_board` in docs, research notes, sandbox suites, `mill-config.yaml`, comments and tests becomes `_bolt`.
  This includes the `_boardroom` near-miss case in `TestIsReservedHubName`, now `_boltroom`.
- `tools/boltrename` is new: a one-shot program that renames an existing hub's checkout.
  Run it once per hub with `go run ./tools/boltrename -hub <hub dir>`, after a lym binary that knows only `_bolt` is installed.

## Design decisions

- lym itself knows `_bolt` only.
  There is no fallback to `_board`, no detection of the old name and no migration case; `boltrename` is the only place the old name appears.
- `boltrename` handles the hubs on Neve and OSL-1046 as they are today.
  It stops and names what it found when the hub holds anything but a prime, its weft, `_launchers`, the Bolt checkout and workspace files, or when the checkout's git worktree links are not the ones it expects.
- It refuses while a lym process, or any process working in the hub, is alive.
- It renames the checkout and its git worktree admin directory, rewrites the two gitdir links between them, moves the hub's own process records, and re-points every `_board` path in the hub's workspace files, the prime's `.vscode` files and the weft's `reed.json`.
- It never runs git.
- The two bulk data files `calls.tsv` under `docs/research` change only where they quote the old path.
- No refactor rides along.
- The task runs alone, with no other run in flight.

`boltrename` is deleted from the repository once both hubs are migrated.

## To check by hand

- Run `boltrename` on a copy of a real hub (Neve, then OSL-1046) and confirm lym starts and `lym board` reads the board from `_bolt`.
- Confirm no live reed command, VS Code setting or workspace file still holds `_board` after the run.
- Search the tree for `_board` outside `tools/boltrename`; none should remain.
