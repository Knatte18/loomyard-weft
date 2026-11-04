# lym ide opens configured files on start, rendered markdown for a beam README

`ide-open-files` · Task · enhancement, ide, config · running · Worktree-Create

## Seen (operator, 2026-11-03)

A beam's VS Code window opens with no file; the operator wants it to start on the README, rendered, not as source.
The operator renders markdown by hand today with a plugin's `Ctrl+K V`.
Later each beam will have markdown files of its own, which the window should open instead.

## Idea (operator, 2026-11-03)

- Which files `lym ide spawn` opens on start is config, per place (prime, beam, task), never hard-coded: a list of paths relative to the worktree, with placeholders such as `{beam}` so a beam's own file can be named later.
- The default opens `README.md` for a beam.
- A markdown file opens rendered.

## Hints (hub orch)

- The `code` CLI opens files given after the folder or workspace path.
- VS Code renders markdown on open through the `workbench.editorAssociations` setting `{"*.md": "vscode.markdown.preview.editor"}`, which lym can write into the seeded workspace settings, with no plugin; `workbench.startupEditor: readme` opens a README rendered when no other editor opens.

Built (2026-11-04): task `beam-code-opens-workspace` makes `lym ide spawn <beam>` regenerate and open the beam's `<RepoName>-<beam>.code-workspace` file, so a beam's `code` button opens the beam workspace; a task slug still opens its bare folder.
The files-on-start idea above is not built.
