---
round: 2
focus:
  - "internal/boardengine/render.go RenderToDisk write loop: does every rendered output path (readme, design_prefix-derived files) pass fsx.PathGuard and get its parent directory created before the in-place os.WriteFile, so a nested path such as views/Home.md renders and a ../ or absolute path is refused without writing outside the board directory?"
  - "internal/boardengine/render_test.go: do the tests cover a nested readme that renders under its subdirectory and an escaping readme that errors and creates no file outside the board dir, without duplicating TestRenderToDiskManifestCleanup or TestRenderToDiskOutputPaths?"
  - "internal/boardengine/boardtest/concurrency_test.go TestConcurrentReadsDuringUpserts doc comment and the RenderToDisk doc comment in render.go: does each state only what the code does after the in-place rewrite, and does the watcher claim name only the evidenced watcher?"
---

The focus targets the fixes to the path guard and the two comment corrections; the rest of the range was settled in round 1.
