# boardengine: rewrite rendered board files in place so VS Code refreshes

An open VS Code markdown preview of a rendered board file (`Home.md`, `README.md`, `project-<slug>.md`, design docs) went stale after a board write.
The cause is in how `boardengine.RenderToDisk` wrote files, not in the workspace settings.
It wrote each file through `fsx.AtomicWrite`, a temp file plus rename, so every render gave the file a new inode.
VS Code's file watcher reports a file replaced by rename as created, not changed, and the markdown preview refreshes only on a change event.

`RenderToDisk` now rewrites an existing output file in place with `os.WriteFile`, keeping its inode, so the watcher reports a change.
`board.json` is still saved atomically.

## Design decisions

- Only the rendered files change write mode.
  They are derived output that the next render rebuilds, so the loss of atomicity costs less than a stale preview.
- A reader that opens a rendered file during a render can now see it half written; `board.json` readers still cannot.
- `RenderToDisk` calls `fsx.PathGuard` on each output path and runs `os.MkdirAll` on its parent directory before writing.
  `AtomicWrite` did both, so this keeps a path escaping the board directory refused and keeps nested outputs such as `views/Home.md` working.
- The workspace settings were not changed.
- The `RenderToDisk` and package docs record the reason, so a later change back to temp-and-rename shows what it breaks.

## Tests

- `TestRenderToDiskRewritesAnExistingFileInPlace` pins that a re-render keeps the file's inode (`os.SameFile`) and the new content lands.
- `TestRenderToDiskOutputPaths` pins that a nested output path is created and that a `../` path is refused with nothing written.
- The comment in `boardtest/concurrency_test.go` now matches the new write path.

## To check by hand

- The cause rests on VS Code's documented watcher behavior; no run can watch VS Code.
  To confirm the fix, open `_bolt/Home.md` (or any `project-<slug>.md`) in the markdown preview, run a board write such as `lym board upsert`, and check that the preview updates without reopening the file.
  Repeat a few times, since the symptom was intermittent.
- Open finding, `RenderToDisk` in `internal/boardengine/render.go`: an earlier version of the write loop dropped the output path guard and the parent-directory creation.
  Check that the loop as it stands calls `fsx.PathGuard` and `os.MkdirAll` before `os.WriteFile`, which the diff shows it does.
