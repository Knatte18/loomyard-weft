# heddle renders claude-file to the worktree root, not to the worktree's anchor

`heddle-render-ignores-anchor` · Note · bug, heddle, fabric, cwd

Imported from [issue #13](https://github.com/Knatte18/loomyard/issues/13).

Seen: in a hub whose worktrees have the anchor `src/csharp/NORCE.Models` (`lym fabric anchor` prints it), `lym heddle render` writes the `claude-file` target to `<worktree>/CLAUDE.local.md` at the git root. The repository's own `CLAUDE.md`, the VS Code workspace folder and every lym session's working directory are the anchor, three levels down, so the operator never sees the rendered file in the workspace, and it sits apart from the `CLAUDE.md` it complements. Claude Code still loads it only because it reads CLAUDE files from every ancestor directory.

Heddle has no setting for the target path (`lym config heddle --resolved` shows only `lym_override`).

Expected: file targets render relative to the worktree's anchor, so `CLAUDE.local.md` lands beside the repository's `CLAUDE.md` in the working directory, or heddle offers a per-target path that defaults to the anchor.

Workaround: none applied; the hub holds its heddle migration until this is fixed.