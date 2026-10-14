# Run window opens as soon as the worktree exists

`run-window-early` · Note · enhancement, batten, reed, fabric

## Seen (operator, 2026-10-14)

`lyx batten run perch-budget-arbiter --window` took about 20 s from start to the Konsole window, and the operator finds that far too slow.
The trace of that start:

- 19:03:20.6: both worktrees, branches, links and launchers created, under 0.1 s in all.
- 19:03:20.6 to 19:03:27.8: three pushes to GitHub, about 1.8 s each, plus a weft commit; `fabricengine` add pushes the warp and weft branches synchronously (`internal/fabricengine/add.go`, steps 11 and 12), and batten pushes the weft again after its own commit.
  Only `lyx fabric sync` pushes async.
- 19:03:29.5 to 19:03:39.4: batten waits for the inner shed run to spawn, and only then spawns the terminal window.

## Direction (operator, 2026-10-14)

Start Konsole with `lyx reed attach` in the pair as soon as the warp worktree is up, before the pushes, the weft seed and the inner shed spawn; `lyx reed attach` boots the reed session itself when it does not exist yet (operator).

## Scope

This task moves the window's start only; the pushes stay as they are.

## Out of scope, for a later note

- Whether the add's pushes can go async like `fabric sync`, or be coalesced into one push per side; a push failure today rolls the add back, so an async push needs another failure path.
