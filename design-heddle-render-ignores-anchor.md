# heddle renders claude-file to the worktree root, not to the worktree's anchor

`heddle-render-ignores-anchor` · Task · bug, heddle, fabric, cwd · running · Code-Burler-Fix

Imported from [issue #13](https://github.com/Knatte18/loomyard/issues/13).

Seen: in a hub whose worktrees have the anchor `src/csharp/NORCE.Models` (`lym fabric anchor` prints it), `lym heddle render` writes the `claude-file` target to `<worktree>/CLAUDE.local.md` at the git root.
The repository's own `CLAUDE.md`, the VS Code workspace folder and every lym session's working directory are the anchor, three levels down, so the operator never sees the rendered file in the workspace, and it sits apart from the `CLAUDE.md` it complements.
Claude Code still loads it only because it reads CLAUDE files from every ancestor directory.

Heddle has no setting for the target path (`lym config heddle --resolved` shows only `lym_override`).

Expected: file targets render relative to the worktree's anchor, so `CLAUDE.local.md` lands beside the repository's `CLAUDE.md` in the working directory.

Workaround: none applied; the hub holds its heddle migration until this is fixed.

## Design (operator, 2026-11-02)

Read from the source: the file path of a target is the literal `File` of a row in `targetTable` (`internal/heddleengine/target.go`), joined to the render request's `Root` in `render.go`; it is no config key.
No config key is added: file targets always render under the worktree's anchor, and in a hub whose worktrees have no anchor the anchor is the root, so such a hub renders as before.

Moving the join from the root to the anchor is not enough, since heddle assumes the root is the place in more than one spot:

- the check that decides whether the `claude-file` target imports the `AGENTS.md` files, which looks for `CLAUDE.md` and `.claude/CLAUDE.md` at the root, and the codex-file target's opening from the root's `AGENTS.md`, must look at the anchor, where the repository's own files sit;
- each target's `ReRender` list holds root-relative paths whose change re-renders the target, and must name the same paths under the anchor;
- the paths fabric's exclude list names for the rendered files must follow, or the file shows as untracked in git;
- the render's temporary directory may stay at the root, since a rename moves each file onto its target.

The writer reads how the render request's `Root` is set by its callers, and where the anchor is already available there, before choosing where the anchor is resolved.
Add tests for a worktree with an anchor and for one without.
