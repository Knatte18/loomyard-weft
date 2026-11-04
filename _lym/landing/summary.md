# lym ide opens configured files on start, markdown rendered

A beam's VS Code window now starts on its `README.md`, rendered, instead of an empty editor.
Which files open is config, not code, so a beam's own markdown files can be named later without a code change.

## What changes

- A new hub-wide `ide.yaml`, owned by the new `internal/ideconfig` package and registered in `configreg`, holds two keys:
  - `open_files`: one ordered list per place (`prime`, `beam`, `task`) of paths relative to the worktree's anchored directory.
    `{beam}` stands for the beam's slug; for a task it is the beam the task was forked from, or the prime's name for a task forked from the prime.
    The template opens `README.md` for a beam and nothing for the prime or a task.
  - `render_markdown` (on in the template): whether a markdown file opens rendered.
- `lym ide spawn` and `lym ide open` pass those files to `code` after the folder or workspace file.
  `vscode.Launch` and the `CodeLauncher` seam now take the target plus the files to open beside it.
  An entry that names no regular file is skipped and logged, since VS Code would open an empty editor on a missing path.
- With `render_markdown` on, the seed sets the `*.md` entry of `workbench.editorAssociations` in the worktree's `settings.json` to `vscode.markdown.preview.editor`, so no plugin is needed.
  Off removes that entry only while it names the preview, so an editor the operator chose stays.
- `docs/overview.md`, the `ideconfig` package doc and `PATTERN-config-layers.md` (degrading callers) are updated.

## Design decisions

- `ideconfig.LoadConfig` degrades to the template through `LoadOrTemplate`, so opening a window never fails on a hub not yet reconciled.
  A malformed `ide.yaml` still fails the spawn, seed or open before any launch.
- `LoadConfig` refuses an `open_files` entry that is empty, absolute or climbs out of the worktree, naming the key.
- Both keys change only what a window shows, so they are `CategoryNone` and a worktree's local override may set them.
  The file is read over the config stack of the spawned slug's own worktree, not the caller's.
- `ideconfig` is its own package, not part of `ideengine`, because `configreg` registers its template and the engine already reaches `configreg` through its imports.
- `lym` owns only the `*.md` entry of `workbench.editorAssociations`.
  A null value counts as absent; the array form of the setting is left untouched, since lym cannot set its entry without replacing the operator's.
- A prime whose name cannot be resolved now takes the task path with its files, instead of returning early with none.

## To check by hand

- Nothing here launched a real VS Code window: run `lym ide spawn <beam>` on a beam with a `README.md` and confirm the window opens it rendered, and that a second `lym ide open` on a task opens its files.
- `PATTERN-config-layers.md` now lists `ideconfig` among the degrading callers; confirm no other list of config-module callers omits it.
- The `ide.yaml` load-failure path (`LoadConfig` error through `Seed`, `Spawn` and `OpenTask`) is thinly tested: check that a malformed file fails each entry point before a launch.
- The prime path of `open_files` is thinly tested: check the prime's list opens beside the hub workspace file.
- The `{beam}` placeholder for a task and `OpenTask`'s files are thinly tested: check a task forked from a beam and one forked from the prime each resolve `{beam}` as described above.
