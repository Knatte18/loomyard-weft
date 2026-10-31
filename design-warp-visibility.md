# warp-visibility: heddle, a Bolt database lym renders its instruction files from, leaving no trace in warp

`warp-visibility` · Note · feature, ide, fabric

## Seen

Using LoomYard on a repository should be invisible: nothing lym-related in the repository's own git history, and no lym text in a file the repository's own people read (operator, 2026-10-29).
Some repositories, Models among them, are restrictive without a `CLAUDE.md`, so lym's agents still need an instruction file of lym's own beside the team's (operator, 2026-10-31).

## Design (operator, 2026-10-31)

LoomYard puts no `CLAUDE.md` into a repository; a repository's own `CLAUDE.md`, if its people added one, is read as usual.
PATTERN.md stays tracked in warp: it holds rules for all development in the repository, by hand or by any agent, not only LoomYard's.

Lym renders its own instruction files into each worktree from one database in Bolt, named heddle:
- `_board/heddle.json` is a JSON database beside `board.json`, and the only source of these files.
  As with the board, the data is kept apart from how it is shown, so it can be rendered and handled in any number of ways later.
- An entry is one instruction passage: a slug, its text, its place in the order, and the files it renders into, such as `CLAUDE.local.md`, Codex's own file, or both; possibly also when it renders.
  Claude and Codex need different text, so an entry names its targets rather than every entry going to every file.
  The same database can later render other per-worktree files, such as a `PATTERN.local.md` should PATTERN ever leave warp.
- Rendering a target joins every entry that names it, in order.
- The database is written only through `lym heddle` verbs, under the write lock of `internal/lock` as the board's store is; never by hand.
- Each worktree lym makes gets the rendered files seeded into its warp, ignored through `.git/info/exclude`, never `.gitignore`, so warp alone carries no trace of LoomYard or of the engines used.
- An entry holds only what is specific to using LoomYard on the repository: its text goes into every agent that runs, so anything that fits PATTERN, a stencil, a skill or a hook goes there instead.

## Open

- Which file Codex reads for local instructions that stays out of the team's history, to be checked.
- When rendering happens: at worktree creation only, or also into live worktrees when the database changes.
- Whether a heddle commit needs an entry-level replay in fabric as `board.json` has, or whether its rare writes make a plain git conflict acceptable.
- The verbs of `lym heddle`.
