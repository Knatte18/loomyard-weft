# An old lyx process drops board fields it does not know

`board-field-drop` · Note · bug, board, batten

## Seen (2026-10-12)

`board-entry-age` landed and deployed new entry fields (`created`, `revised`), and a backfill wrote them for every entry.
Within minutes a status write from a run dropped every date: board commit `5e5402f5` in the board repo rewrote `board.json` without the new fields, and the same happened again after a second backfill.
The writers were long-lived `lyx batten run` and `lyx shed step --until-stop` processes started before the deploy: the old binary decodes an entry into its own struct, so a field it does not know is lost when it writes the board back.

## Why it matters

Any deploy that adds a board field loses that field's data while a run started before the deploy is still writing status, and runs live for hours.
The same holds for any other record file a long-lived process rewrites whole.

## Idea

- A board writer keeps fields it does not know: it decodes entries so unknown keys survive a rewrite, or a status write patches only `status` instead of rewriting the entry.
- Or a writer refuses to write a board whose schema is newer than it knows, naming the restart it needs.
