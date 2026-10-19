# [WS 002] One VS Code workspace per umbrella, listing its worktrees directly

`hub-workspaces` · Task · enhancement, fabric, ide, workspaces · running · Merge-In

## Seen (2026-10-18)

- Fabric writes one hub workspace for the prime, `<hub>/_launchers/<AnchorRel>/<primeName>.code-workspace` (`internal/fabricengine/hubworkspace.go`), with the folders prime, `_board` and `_portals`.
- `_portals` holds one link per live pair, pointing at that pair's `_lyx` only, so the workspace file never changes as pairs come and go, but a pair's code is not in it.
- Opening it in VS Code works, and the operator found it very promising.
- A stale prime `.vscode/tasks.json` from before `6667f79df` auto-ran `reed up`, `reed add claude` and `reed attach` on folder open; it was git-ignored leftover state and is deleted.

## Operator decisions (2026-10-18)

- Workspaces are how the operator opens a hub: one workspace per umbrella (see `orch-any-worktree`); today only the prime, the root umbrella.
- Every folder entry points straight at the worktree's anchor directory, so the operator never navigates down to the right subdirectory.
- The umbrella's own worktree is always the first folder, then Bolt, then every worktree belonging to the umbrella, each listed directly; `_portals` is not needed.
- A finished pair's folder is removed from the workspace again.
- Folder display names: the prime's worktree shows as its branch name (`main`), `_board` shows as `Bolt`, a task as its slug; directories on disk keep their names.
  The prime's directory kept the repository's name only because VS Code's recent-folders list showed many indistinguishable `main` folders, which workspaces make moot.
- The workspace files live directly in the hub directory, ready to open; the hub directory is untracked, as `_launchers` and `_portals` are today.

## Operator decisions (2026-10-18, later)

- Each file is named with the repository in it: `loomyard.code-workspace` for the root umbrella and `loomyard-<umbrella-slug>.code-workspace` for another, since VS Code's recent list shows workspaces by file name.
  An umbrella is named by its slug on the board.
- Folder order: the umbrella's own worktree first, then Bolt, then the other worktrees alphabetically by display name, so a task is easy to find.

## Orch recommendation, not a decision

- Fabric rewrites an umbrella's workspace at every `add` and `remove` of a pair under it; VS Code picks up a changed folder list in an open window, and only a change of the first folder restarts its extensions, which never happens since the umbrella stays first.
  A drag in VS Code rewrites the file and lasts until fabric's next rewrite.
