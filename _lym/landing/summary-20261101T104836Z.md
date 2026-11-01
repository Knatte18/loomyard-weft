# Rename the Bolt checkout directory _board to _bolt

The hub's Bolt checkout holds the board, the hub's `_lym` config, orch rules and heddle's database, so its directory is now named after the repository it checks out: `_bolt`, not `_board`.
lym knows only the new name; a hub made before this change is renamed once by a program shipped for that purpose.

## What changes

- `lymcwd.BoltDirName`, the one declaration of the name, is `"_bolt"`; every path lym builds into the checkout follows it.
- Every other `_board` spelling in the tree is now `_bolt`: Go tests, the geometry enforcement tables, `docs/`, `tools/sandbox` suites, the overview SVG, research records and a recorded transcript in the claudeengine corpus.
  The swap was made by script: an occurrence of `_board` with no letter or digit before it became `_bolt`.
  Identifiers and test names that say `board` for the board itself (`custom_board`, `no_board_path_resolves_via_paths`) are unchanged; the near-miss fixture `_boardroom` became `_boltroom`, and the two enforcement case names that spelled the token follow it.
- `tools/boltrename` is the per-hub rename program, run as `go run ./tools/boltrename -hub <hub dir>`.
  It refuses while any process runs a `lym` binary, works in the hub, or names the hub on its command line, listing them.
  It stops and names what it found when the hub holds anything but a prime, its weft, `_launchers`, `_board` and workspace files, or when `_board/.git` and the weft's `.git/worktrees/_board/gitdir` do not point at each other.
  Otherwise it renames `_board` and its git admin directory, rewrites both gitdir links, moves the hub's own process records under `.lym/runproc`, and re-points `_board` paths in the hub's `*.code-workspace` files, the prime's `.vscode` files and the weft's `.lym/reed.json`.
  It never runs git.

## Design decisions

- **No compatibility in lym.** Nothing in lym looks for `_board`, falls back to it or migrates it; `tools/boltrename` is the only place the old name is spelled, and is to be deleted once the hubs on Neve and OSL-1046 are renamed.
- **The program handles the hubs as they are, with no what-if code.** It acts on the layout and the files found on Neve, and stops on anything else, leaving that to be fixed by hand.
  It does not walk the hub for other mentions: logs, gate run records, creel journals and the weft's tracked run records keep `_board` as history.
  A pair still in the hub counts as unexpected, so the program also stops while any run's worktree is present.
- **The git worktree link is fixed by file, not by `git worktree move`.** The admin directory is renamed with the checkout and both gitdir files are written directly, which is what `git worktree move` would leave, and keeps the tool outside the git boundary rules.
- **Bolt's own contents are not in this diff.** The Bolt copies of stencils and specs take the new text through the existing stencil refresh; board entries and design files that mention `_board` in prose are not rewritten.

Closes nothing: the board entry lists no issue.

## To check by hand

- On each hub, with every lym process, agent and editor in it ended and the new binary installed: `go run ./tools/boltrename -hub <hub dir>` prints the rename and each re-pointed file; then `git -C <hub>/_bolt status` is clean and `lym board list` works.
- `internal/lymcwd/anchor.go:25` declares `BoltDirName = "_bolt"`, and `TestEnforcement_BoltDirNameDeclaredOnce` and `TestEnforcement_GeometryLiterals` in `internal/lymcwd/enforcement_test.go` police `_bolt`.
- `tools/boltrename/rename_test.go` pins a full rename of a hub laid out as Neve's, each stop on an unexpected hub, and which processes block the run.
- `git grep -n _board` finds only `tools/boltrename` and the identifiers and test names that say board for the board.
- Claude Code's own per-path records, such as a trusted-folder entry in `~/.claude.json` for `<hub>/_board`, are not re-pointed; none existed for the Neve hub's checkout.
