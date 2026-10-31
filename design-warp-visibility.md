# warp-visibility: heddle, a Bolt database lym renders its instruction files from, leaving no trace in warp

`warp-visibility` · Task · feature, fabric, board, config, claude, codex, cwd, process, stencil · draft

Depends on: [`bolt-dir-rename`](design-bolt-dir-rename.md)

## Seen

Using LoomYard on a repository should be invisible: nothing lym-related in the repository's own git history, and no lym text in a file the repository's own people read (operator, 2026-10-29).
Some repositories, Models among them, are restrictive without a `CLAUDE.md`, so lym's agents still need an instruction file of lym's own beside the team's (operator, 2026-10-31).

## Design (operator, 2026-10-31)

LoomYard puts no `CLAUDE.md` into a repository; a repository's own `CLAUDE.md`, if its people added one, is read as usual.
PATTERN.md stays tracked in warp: it holds rules for all development in the repository, by hand or by any agent, not only LoomYard's, and its `(test)` entries are enforced by tests that live in warp, so a developer without LoomYard must see the rule a failing test stands for.
This task takes PATTERN into heddle in no form, but builds heddle so that a further target such as `PATTERN.local.md` is easy to add later.

Lym renders its own instruction files into each worktree from one database in Bolt, named heddle:
- `heddle.json` is a JSON database in the Bolt checkout beside `board.json`, and the only source of these files.
  As with the board, the data is kept apart from how it is shown, so it can be rendered and handled in any number of ways later.
- An entry is one instruction passage: a slug, its text, its place in the order, and the files it renders into, such as `CLAUDE.local.md`, Codex's own file, or both; possibly also when it renders.
  Claude and Codex need different text, so an entry names its targets rather than every entry going to every file.
- Rendering a target joins every entry that names it, in order.
- Codex reads no `AGENTS.local.md`: in each directory it loads one file only, `AGENTS.override.md` before `AGENTS.md` before the names in `project_doc_fallback_filenames` (OpenAI's AGENTS.md guide, checked 2026-10-31).
  So heddle renders Codex's target as `AGENTS.override.md`: the repository's own `AGENTS.md`, when it has one, followed by lym's entries for Codex, so the team's instructions are not hidden (operator, 2026-10-31).
  A team edit of `AGENTS.md` reaches a worktree when a fabric sync finds commits on the remote that the worktree lacks and settles them locally; that is when heddle renders `AGENTS.override.md` again (operator, 2026-10-31).
  Fabric knows nothing of heddle: it declares a general seam for "warp took in remote commits, and these paths changed", which heddle implements and the lym root pre-run installs, the way `InstallBoardReplay` and `InstallPushNotifier` are installed today; a sync running detached reaches it too, since that process is lym with the same pre-run.
- The database is written only through `lym heddle` verbs, under the write lock of `internal/lock` as the board's store is; never by hand.
- Each worktree lym makes gets the rendered files seeded into its warp, ignored through `.git/info/exclude`, never `.gitignore`, so warp alone carries no trace of LoomYard or of the engines used.
  Fabric already seeds `.git/info/exclude`, anchored and idempotent, with an unseed beside it, but only for the names it makes junctions of (`junctionNames`, `seedGitExclude`).
  The excludes become a list of their own in fabric's config, separate from the junction list: fabric seeds every path on it when it creates a warp and weft, through the same seeding, and heddle's target files are put on that list in config, so fabric names no heddle file (operator, 2026-10-31).
- An entry holds only what is specific to using LoomYard on the repository: its text goes into every agent that runs, so anything that fits PATTERN, a stencil, a skill or a hook goes there instead.

- A worktree's file is rendered again only when something in that worktree changed that makes re-rendering that very file meaningful, such as a sync that took in a new `AGENTS.md`; a change to `heddle.json` is not pushed into live worktrees, so an agent's rules do not shift mid-run (operator, 2026-10-31).
- No one edits `AGENTS.md` by hand inside a worktree, so heddle watches for no such edit (operator, 2026-10-31).
- Two machines writing `heddle.json` at once are settled by the entry-level replay `board.json` already has, reused and not written again (operator, 2026-10-31).
  Fabric holding a seam tailored to the board makes no sense, so the task makes it general (operator, 2026-10-31):
  - fabric's seam (`fabricengine.BoardReplay`, its request and result types, `InstallBoardReplay`, and the board paths `replayBolt` recognises) is renamed and reshaped so that no board name or board path is left in fabric;
  - fabric knows only that a file in Bolt has an entry-level replayer registered for it, walks the local commits and cherry-picks the rest as today;
  - the board and heddle each register a replayer for their own file;
  - the entry-by-entry merge in `boardengine`'s `FabricReplay` that both need is drawn out and shared, not copied.
- A re-render after a sync takes lym's text from `heddle.json` as it stands then; a worktree keeping the version it was created with belongs to a separate note and is not this task's (operator, 2026-10-31).
- The verbs of `lym heddle` are the task's to shape.

## Bolt directory name (operator, 2026-10-31)

The rename of `_board` to `_bolt` lands first as a task of its own; this task then makes the name live in one place, since heddle's database takes its place from it.
- Two names for two things: `fabricengine.BoltDirName` says where Bolt is checked out, and `fabricengine.BoardDirName` says where the board is.
  Today the board sits at Bolt's root, so `BoardDirName` is set from `BoltDirName`; what lives in Bolt beside the board, heddle's database included, takes its place from `BoltDirName`, never from `BoardDirName`.
- `runproc.HubOwner` is an owner name, not the directory, and is set from `BoltDirName`.
- The directory name lives in that one constant only: `lymcwd` drops its own copy, and every message and test takes the name from the constant.
- A comment names "the Bolt checkout" rather than the path; stencils and skills name the path through what lym already resolves for them, or as "the Bolt checkout", not as a literal.
