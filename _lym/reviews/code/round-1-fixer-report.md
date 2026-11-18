---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

## Changes

- F1: `RenderToDisk` in `internal/boardengine/render.go` now calls `fsx.PathGuard(relPath)` and `os.MkdirAll` on the parent directory before the in-place `os.WriteFile`, wrapping each error as `write <relPath>`.
  `TestRenderToDiskOutputPaths` in `internal/boardengine/render_test.go` adds a nested `views/Home.md` row and a `../outside.md` row that must error and write nothing outside the board directory.
- F2: the `TestConcurrentReadsDuringUpserts` doc comment says the write saves `board.json` atomically and then rewrites the rendered board files in place.
  A fork found the first rewording over the 120-character line limit, so it is two sentences.
- F3: the `RenderToDisk` doc comment names VS Code's file watcher.
- F1 and F3 share one commit, since both edit `render.go`.

The `internal/boardengine` and `internal/boardengine/boardtest` gates pass.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: LOW, the F2 comment line over the line limit; fixed as proposed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
