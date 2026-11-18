# Rewrite rendered board files in place so a VS Code preview refreshes

An open board file in VS Code's markdown preview often kept showing old content after a board write.
The cause is in how lym wrote the files, not in VS Code's settings.
`boardengine.RenderToDisk` now rewrites each rendered board file in place.

## Cause and evidence

All of the evidence below comes from the installed VS Code 1.142.0 (snap revision 280) and this hub.

- **lym replaced each file by rename.**
  `RenderToDisk` wrote every output through `fsx.AtomicWrite`, which writes a temp file in the same directory and renames it over the target.
  Every board write re-renders every file, since README and project files carry ages and the render time.
- **VS Code reports a rename-over as a creation.**
  On Linux a rename onto an existing file raises inotify `IN_MOVED_TO` on the target, which the parcel watcher reports as `create`.
  `watcherMain.js` maps it through `MAP_PARCEL_WATCHER_ACTION_TO_FILE_CHANGE = [["create",1],["update",0],["delete",2]]`, so the event is `ADDED`, not `UPDATED`.
- **The markdown preview refreshes only on a change.**
  The preview in `extensions/markdown-language-features/dist/extension.js` refreshes on `onDidChangeTextDocument`, `onDidOpenTextDocument`, or its own watcher's `onDidChange`.
  It never subscribes to `onDidCreate`.
  The watcher's `onDidChange` is consulted only when no text document of the file is open.
- **The text document goes away after three minutes.**
  The preview opens the file as a text document, which the workbench reloads on `UPDATED` or `ADDED` (`e.contains(t.resource,0,1)`).
  So the preview refreshes at first.
  But a document an extension opens without an editor is held by a `BoundModelReferenceCollection` with a three-minute maximum age (`1e3*60*3`), then released.
  From then on only the watcher's `onDidChange` can refresh the preview, and a rename-over never raises it.
  This explains why a preview updates soon after it is opened and goes stale later.

Ruled out:

- `files.watcherExclude` in `loomyard.code-workspace` excludes only `**/_lym/**`, and the board files sit at `_bolt`'s root.
  The user settings set no watcher or markdown preview key.
- `_bolt` is a real directory in the hub, not a symlink or junction.
- No setting makes the preview listen for created files, so this cannot be fixed in the workspace file.

## The change

- `RenderToDisk` writes each output with `os.WriteFile`.
  This truncates and rewrites an existing file, keeping its inode, so the watcher reports `update` and the preview refreshes.
  A new file is created with mode `0600`, the mode `os.CreateTemp` gave the old writes.
- The render manifest and the decision, Sketch and raddle files keep their atomic writes.
  lym parses those files back, and they change rarely.
- The `boardengine` package doc and `RenderToDisk`'s doc state the in-place write and why.

## Design decisions

- The rendered `.md` files are a view derived from `board.json`, and lym never parses them back.
  So a reader that catches one mid-write sees a partial view that the next event fixes, and a crash mid-write leaves a view the next render rebuilds.
  A board write renders under the board's file lock, after `board.json` is saved, and its git sync starts only after the render returns.
- Files whose content is unchanged are still rewritten; skipping them is not needed for the fix.

## Tests

`TestRenderToDiskRewritesAnExistingFileInPlace` renders twice with a changed body.
It asserts with `os.SameFile` that the design file is the same file after the second render, and that it holds the new body.

## Open risks

- A git checkout or pull in `_bolt` that rewrites a board file, such as a sync from another machine, was not examined.
  It may still leave an open preview stale.
- Decision record files still go through temp-and-rename, so a preview of one can go stale after an edit.

## How to confirm by hand

1. Install a `lym` built from this change.
2. Open `_bolt/README.md` or a `design-*.md` in the workspace's markdown preview, and leave it open for more than three minutes.
3. Make a board write that changes that file, for example `lym board` editing an entry's title shown in it.
4. The open preview shows the new text within about a second, with no reopening.
   Before this change, the same steps leave the old text in place.
