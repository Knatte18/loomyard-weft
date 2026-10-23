# Show the prime's owner in the board README and lyx board list

The machine that owns the hub's prime was recorded only as `prime_owner` in `board.json`.
The README showed each task's owner but not the prime's, and no `lyx board` verb printed it.
An operator adopting tasks learnt only from `lyx adopt`'s warning that another machine still owned the prime.
The prime's owner is now visible wherever task owners are.

## What changes

- The rendered board README states `Prime owner: ` and the machine in a code span after the render time, or `none recorded` when `board.json` records none.
- `lyx board list` JSON is now `{"ok":true,"prime_owner":"<machine>","tasks":[…]}`.
  The key is always present and empty when none is recorded.
- `lyx board list --text` opens with `prime owner: <machine>`, or `prime owner: none`, followed by the unchanged entry lines.
- `find`, `get` and `list-full` are unchanged, and so is how the prime owner is recorded or replayed.
  The `find` help no longer claims list's full JSON; it says it prints the same tasks array in an envelope without `prime_owner`.
- The `internal/boardengine` and `internal/boardcli` package docs and the board skill's list paragraph describe the new output.

## Design decisions

- `Render` and `RenderToDisk` take the prime owner beside the tasks, so the renderer stays a pure function of its inputs.
  Every caller (board write, owner set, replay) passes its loaded store's `prime_owner`.
- `list` reads the owner through a new `Board.PrimeOwner` beside `ListTasksBrief`, which keeps that function's signature and callers untouched.
  The second load happens under the swap lock, so a write landing between the two reads can only show a fresher owner.
- `Board.PrimeOwner` has no absent-directory branch, because `Store.Load` already reads an absent board as empty.
- The shown value is the recorded `prime_owner`, not a resolved one, since the prime has no parent to inherit from.
- The `board.json` shape is unchanged.

## Check by hand

- A script parsing `list --text` line by line now sees one extra first line; no consumer in the repository does.
- The README prime-owner line is pinned at each render site (board write, owner set, replay) by `TestPrimeOwnerReadAndRendered` and the owner-set and replay tests; confirm the line shows the right machine on a real hub's README after a write.
- `Board.PrimeOwner` on an absent board directory returns empty; this is pinned in the engine test only, with no `lyx board list` run against a missing board directory.
- The `find` help wording in `internal/boardcli/cli.go` matches its actual output.
