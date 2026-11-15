# A beam window opens no README by default

The `ide.yaml` template in `internal/ideconfig` listed `README.md` under `open_files.beam`.
The `code` button in a beam therefore opened the beam worktree's README, which is of no use there.
The template now sets `open_files.beam: []`, so no place (prime, beam or task) opens a file unless a hub adds an entry.
A hub that wants a beam file, such as `docs/{beam}.md`, names it in its own `ide.yaml`.

Existing hubs that kept the old template's `README.md` entry in their `ide.yaml` still open it; they must remove the entry.

The change also updates the `ideconfig` package doc, `docs/overview.md` and the `TestLoadConfig` expectations to match.

## Design decisions

- `render_markdown` and its `workbench.editorAssociations` `"*.md"` entry in `internal/vscode/config.go` are unchanged.
  The report that the markdown preview does not render in a lym-opened window was not diagnosed here.
  It showed while VS Code had an Extension Bisect running, which may have disabled the markdown extension.
  Whether lym's settings are at fault is still open.

## To check by hand

- Open a window with `lym ide spawn` or `lym ide open` on a beam and confirm that no file opens.
- Open a `.md` file in a lym-seeded window with no Extension Bisect running, and confirm whether it renders as the preview.
  If it does not, `render_markdown` needs a follow-up task.
