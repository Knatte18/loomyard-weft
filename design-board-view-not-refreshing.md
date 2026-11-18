# Find why VS Code keeps showing a stale board file, and fix it

`board-view-not-refreshing` · Task · bug, board, ide · running · Drift-Expect

## What was seen (operator, 2026-11-18)

VS Code often does not update the board file the operator is looking at after a board write.
The files sit in `_bolt`, a folder of the hub workspace.
The workspace opens every `.md` in the markdown preview editor (`workbench.editorAssociations` in `loomyard.code-workspace`).

## Operator decision (2026-11-18)

Find out why first; do not assume a cause.
The operator expects the cause in VS Code's settings, not in LoomYard.

## Scope

1. Find the cause and state the evidence in the report.
   Candidates to check, none confirmed:
   - workspace settings such as `files.watcherExclude`, `files.watcherInclude` or the markdown preview settings;
   - how the workspace reaches `_bolt` (a symlink or junction, which VS Code's file watcher may not follow);
   - the board's temp-and-rename writes in `boardengine.RenderToDisk`, which give each rendered file a new inode.
   Use VS Code's own documentation and issue tracker for how its watcher and the markdown preview behave.
2. Fix it where the cause is.
   A settings fix goes into the workspace file or the code lym uses to write it.
   Change how lym writes board files only if the cause is there.
3. Say in the report how the operator can confirm the fix by hand, since a run cannot watch VS Code.
