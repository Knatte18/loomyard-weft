# A beam window opens no README by default

The `code` button in a beam opened the beam worktree's `README.md`, which says nothing about the beam.
`ide.yaml`'s template listed `README.md` under `open_files.beam`.
The template now sets `open_files.beam: []`, so a window lym opens shows no file by default in any place.
A hub that wants a file opened names it by adding an entry, for example `docs/{beam}.md`.
A hub that already set `beam: []` in its own `ide.yaml` behaves as before.

## Markdown preview: settings left unchanged

The task also reported that a `.md` file did not render as a preview in a window lym opens.
The cause was not found in lym's settings, so `render_markdown` and the association it writes stay as they are:

- `vscode.markdown.preview.editor` is the custom editor that the built-in `markdown-language-features` extension registers for `*.md` in the installed VS Code (1.142).
  So the association lym writes names a valid editor.
- The association lands where VS Code reads it.
  The prime's and a beam's workspace files carry it under `settings`, and a task folder's `.vscode/settings.json` carries it too.
  `workbench.editorAssociations` is window-scoped, so workspace settings apply it.
  No user setting overrides it.
- Extension Bisect disables only user-installed extensions.
  The stored bisect state lists ten user extensions and not the markdown extension.
  So bisect could not have switched off the preview editor either.

The preview was not checked in a live VS Code window.
If it still fails once bisect has ended, the next step is to look at VS Code's own state, such as its editor-resolver cache, and not at lym's settings.

## Changes

- `internal/ideconfig/template.yaml`: `open_files.beam` is `[]`.
- `internal/ideconfig/doc.go` and `docs/overview.md` state that the template opens nothing for any place.
- `internal/ideconfig/config_test.go`: the template-default and local-override rows expect an empty beam list.
