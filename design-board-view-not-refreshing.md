# VS Code often keeps showing a stale board file after lym rewrites it

`board-view-not-refreshing` · Note · bug, board, ide

## What was seen (operator, 2026-11-18)

VS Code often does not update the board file the operator is looking at after a board write.
The files sit in `_bolt`, which is a folder of the hub workspace and not under the watcher-excluded `**/_lym/**`, so the watcher does see the folder.
The workspace opens every `.md` in the markdown preview editor (`workbench.editorAssociations`).

## Likely cause

`boardengine.RenderToDisk` writes each rendered file with `fsx.AtomicWrite`: a temp file renamed over the old one, so each write is a new inode.
An open preview of the old file can miss that replace and keep the old content until the file is reopened.

## Idea

Confirm by hand first (keep a board file open in the preview, run a board write, see whether it refreshes; repeat with an in-place write).
If the rename is the cause: write the rendered view files (README, project and design files) in place, truncate and write, keeping atomic writes for the board's data files.
A torn view file is harmless; the next render rewrites it.
