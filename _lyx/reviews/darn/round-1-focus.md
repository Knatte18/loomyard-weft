---
round: 1
focus:
  - "internal/boardengine render path: does every caller of `Render` and `RenderToDisk` pass the loaded store's prime owner, including any site the summary does not list, and is the empty-owner line the one the board entry implies?"
  - "internal/boardcli `list` verb: is `prime_owner` always present in the JSON envelope, does `--text` open with the prime owner line before the entry lines, and is `find` output left unchanged?"
  - "`Board.PrimeOwner` for an absent board directory and for a recorded owner: does it read the same field `store.go` writes, and is it covered by a test?"
  - "Docs named by CLAUDE.md for observable CLI behavior change: do `doc.go`, `docs/overview.md` if affected, and the `list` help text match the diff's new output?"
---

Seed focus from the board entry's two asks: README states the prime owner at its top, and `lyx board list` prints it in JSON and `--text`.
