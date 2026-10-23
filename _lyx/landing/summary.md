# Board README and lyx board list show the prime owner

The machine that owns the hub's prime was recorded only as `prime_owner` in `board.json`.
The README showed each running task's owner, but not the prime's, and no `lyx board` verb printed it, so an operator adopting tasks learnt who held the prime only from `lyx adopt`'s warning.

## What changes

- **README.** The opening paragraph gains a line under the render time: ``Prime owner: `<machine>`.``, or `Prime owner: none recorded.` when `board.json` records none.
  `Render` and `RenderToDisk` take the prime owner beside the tasks, and every render site passes its loaded store's record: the board write path, the owner setter and the board replay.
- **`lyx board list`.** The JSON envelope carries a top-level `prime_owner` key, always present and empty when none is recorded.
  With `--text` the listing opens with `prime owner: <machine>`, or `prime owner: none`, before the entry lines.
  `find --text` is unchanged.
- **`Board.PrimeOwner`.** A new read that returns the recorded prime owner, `""` for an absent board directory.

## Design decisions

- The prime owner shown is the recorded one, not a resolved one: the prime has no parent to inherit from.
- `list` reads the prime owner through its own `Board.PrimeOwner` call beside `ListTasksBrief`, leaving that method's signature and callers unchanged.
  The two reads are separate store loads; a write landing between them can only show a fresher owner.
- The empty case is stated in the README and the text listing, so the line keeps its place and an absent record reads as a fact rather than an older binary.

## Docs

`internal/boardengine/doc.go` and the `render.go` file header, `internal/boardcli/doc.go`, the `list` verb's `--help`, and the board skill's list paragraph name the new output.

## Tests

- `TestRenderReadmeGolden` pins the new README line for a recorded owner and for none.
- The `TestCLI` steps `Owners` and `ListAndFindText` pin `prime_owner` in the JSON, recorded and empty, and the `--text` opening line, recorded and `none`.
