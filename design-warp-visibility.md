# warp-visibility: lym renders its instruction files from one Bolt store, leaving no trace in warp

`warp-visibility` · Note · feature, ide, fabric

## Seen

Using LoomYard on a repository should be invisible: nothing lym-related in the repository's own git history, and no lym text in a file the repository's own people read (operator, 2026-10-29).
Some repositories, Models among them, are restrictive without a `CLAUDE.md`, so lym's agents still need an instruction file of lym's own beside the team's (operator, 2026-10-31).

## Design (operator, 2026-10-31)

PATTERN.md stays tracked in warp: it holds rules for all development in the repository, by hand or by any agent, not only LoomYard's.
A repository's own `CLAUDE.md`, if it has one, is its team's and is read as usual.

Lym renders its own instruction files into each worktree from one source in Bolt:
- Bolt holds one JSON store of render entries beside the board's, the only source of these files.
  Each entry names the files it renders into, such as `CLAUDE.local.md` for Claude and Codex's own file for Codex, and possibly when it renders.
  Claude and Codex need different text, so an entry names its targets rather than every entry going to every file.
  The same store can later render other per-worktree files, such as a `PATTERN.local.md` should PATTERN ever leave warp.
- The store is written only through `lym` verbs, with the same safe concurrent writes the board's store has; never by hand.
- Each worktree lym makes gets the rendered files seeded into its warp, ignored through `.git/info/exclude`, never `.gitignore`, so warp alone carries no trace of LoomYard or of the engines used.
- An entry holds only what is specific to using LoomYard on the repository: its text goes into every agent that runs, so anything that fits PATTERN, a stencil, a skill or a hook goes there instead.

## Open

- Which file Codex reads for local instructions that stays out of the team's history, to be checked.
- When rendering happens: at worktree creation only, or also into live worktrees when the store changes.
- The verbs that write the store.
