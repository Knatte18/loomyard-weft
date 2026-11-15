# A beam window opens a useless README, and markdown preview does not render

`ide-beam-readme-preview` · Task · bug, ide · running · Darn-Write

## Seen (operator, 2026-11-15)

1. The `code` button in a beam opened `README.md` of the beam's worktree, which is of no use there.
   The cause is `ide.yaml`'s template, whose `open_files.beam` lists `README.md` (`internal/ideconfig`).
   The hub's default layer now sets `open_files.beam: []`; the template still opens it for every other hub.
2. The markdown preview does not work in a window lym opens: a `.md` file does not render, the Bolt README included.
   `render_markdown` sets `settings.json`'s `workbench.editorAssociations` `"*.md"` to `vscode.markdown.preview.editor` (`internal/vscode/config.go`).

## Idea

- Drop `README.md` from the template's `open_files.beam`, so every place opens nothing by default.
- Find why the preview association fails and fix it, or drop `render_markdown` if the association cannot work.
  VS Code showed an "Extension Bisect" in progress at the time, which disables extensions and may have switched off the markdown extension;
  settle whether lym's settings are at fault before changing them.
