# lym ide opens configured files on start, markdown rendered

A beam's VS Code window used to open with no file.
Which files a window opens on start is now config, per place, and a markdown file opens rendered rather than as source.

## What changes

- **New hub-wide config module `ide`** (`internal/ideconfig`, registered in `configreg`).
  `open_files` holds one list per place, `prime`, `beam` and `task`.
  Each entry is a path relative to the worktree's anchored directory, the folder its window shows.
  `{beam}` in an entry stands for the beam's slug: a beam's own, a task's parent's, or the prime's name for the prime and a task forked from it.
  The template opens `README.md` for a beam and nothing for the prime or a task.
  `render_markdown` (default `true`) controls the markdown rendering below.
  `LoadConfig` falls back to the template when the hub has no `ide.yaml`.
  It refuses an empty or absolute entry, or one that climbs out of the worktree, with an `ErrInvalid` error naming the key.
  Both keys are open to a worktree's local override, since they change only what a window shows.
- **`lym ide spawn` and `lym ide open`** (`internal/ideengine`) read `ide.yaml` over the caller's config stack, pick the list for the worktree's place and resolve `{beam}`.
  They pass the files that exist to the launcher, in order, after the folder or workspace file.
  An entry that names no file, or a `{beam}` with no beam to resolve, is skipped and logged at Info, so VS Code never opens an empty editor on a path that does not exist.
  An `ide.yaml` that fails to load now fails `spawn`, `seed` and `open` with that error.
- **`vscode.Launch`** takes the files as variadic arguments and appends them to the `code` call, on both Linux and Windows.
  The `ideengine.CodeLauncher` seam's signature changes to match.
- **`vscode.WriteConfig`** takes a `renderMarkdown` flag.
  When it is on, the `*.md` entry of `workbench.editorAssociations` in the worktree's `settings.json` is set to `vscode.markdown.preview.editor`, keeping every other association.
  When it is off, the entry is removed only while it names the preview, and a map left empty is dropped, so an editor the operator chose stays.
  A beam or prime workspace file splices those settings, so its window renders markdown too.

## Design decisions

- The config is a new hub-wide module rather than a key in `desktop.yaml`, because every `desktop` key is a machine key and the files a window opens belong to the repository, not to the machine.
  It lives in its own leaf-like package, as `hibernateconfig` does, because `ideengine` already reaches `configreg` through its imports.
- Rendering uses VS Code's editor association, as the hints suggested, so no plugin is needed.
  `workbench.startupEditor` is left as is, because the window now opens named files.
- A load failure of `ide.yaml` fails the verb instead of degrading, so an invalid config is fixed rather than silently ignored; an absent file still resolves the template.
- Only `{beam}` is offered as a placeholder; the board entry names it and no other.

## Tests

- `internal/ideconfig`: template defaults, a hub file and a local override setting the keys, and refused absolute, climbing and empty entries.
- `internal/vscode`: the `*.md` association set on a fresh and an existing file, removed when off, and kept when it names another editor.
- `internal/ideengine`: a beam spawn opens its listed files in order, with `{beam}` resolved, an absent entry skipped and the workspace carrying the preview association; a task spawn opens no file under the template.
- `internal/configreg`: the module is hub-wide and both keys are local keys.
