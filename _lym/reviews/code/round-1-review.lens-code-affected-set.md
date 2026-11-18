```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering everything outside the touched lines that the diff's changes force.

**Per change, the set found:**

1. **`RenderToDisk` now writes each rendered file in place with `os.WriteFile`, no longer through `fsx.AtomicWrite`.**
   - Callers: `board.go:192` (`boardCriticalSection`), `replay.go:613`, `ownerset.go:336` (`renderOwnerWrite`). All are covered: none depends on the write's atomicity, inode or temp file, and all hold the board write lock.
   - Readers:
     - `sync.go:60-70` `commitDirty` takes the same `writeLockFile`, so a commit never reads a half-written file. Covered.
     - `replay.go:805-845` `applyDesignFile` reads and writes design files through `AtomicWrite` and calls `RenderToDisk` afterwards. Covered.
     - `isBoardOverview` reads only paths. Covered.
   - Comments describing the write:
     - `render.go:13` file header ("RenderToDisk writes the files…"): neutral. Covered.
     - `doc.go:341` and the `RenderToDisk` doc: updated in the diff. Covered.
     - `boardtest/concurrency_test.go:27` still says each write "performs an AtomicWrite of board.json, Home.md and _Sidebar.md". **Not covered.**
   - `fsx.go`: unchanged and still true of `AtomicWrite`. Covered.
   - `docs/overview.md`: no claim about how board render writes files. The `render_markdown` passage at lines 577-579 stays true. Covered.
   - PATTERN: no entry or background file describes how board render writes files. `PATTERN-lym-files` governs temp files and does not reach this change. No PATTERN changes are declared. Covered.
2. **New test `TestRenderToDiskRewritesAnExistingFileInPlace`.**
   - The `render_test.go` file header (lines 1-7) lists what the file covers, the `RenderToDisk` manifest cleanup included, but not the in-place rewrite. **Not covered.**
3. **Commit messages (28c270f4, b05e3b1d).** Neither carries a `Pattern-Departure:` or `Loosened-Check:` trailer, and neither is needed: no PATTERN entry forbids the change and no check was loosened. Covered.

**Findings:**

- **LOW · consistency** · `internal/boardengine/boardtest/concurrency_test.go:27`
  - The doc comment still says each write "performs an AtomicWrite of board.json, Home.md and _Sidebar.md". After the diff, Home.md and the other rendered files are rewritten in place, and only board.json and the manifest still go through temp and rename.
  - Evidence: `concurrency_test.go:27`; `render.go:49`.
  - Fix: reword the line so it names AtomicWrite only for board.json and says the rendered files are rewritten in place.
- **NIT · consistency** · `internal/boardengine/render_test.go:1-7`
  - The file header lists what the file covers, including `RenderToDisk`'s manifest cleanup. It does not mention the new test of in-place rewriting, so the header no longer describes the file's contents in full.
  - Evidence: `render_test.go:6-7` and the new test at diff line 45.
  - Fix: add a clause saying a re-render keeps each output file's inode.

Tool calls:
1. Searched boardengine, docs and pattern for "atomic" and in-place wording, and read `concurrency_test.go:20-35`. The manifest had neither the docs that describe board writes nor that test comment.
2. Searched `concurrency_test.go` for reads of rendered files, and `docs/overview.md`, `PATTERN.md` and `pattern/` for preview or watcher text. Needed to check whether any test or doc depends on rendered files being written atomically.
3. Read `docs/overview.md:572-584`. Needed to confirm the `render_markdown` passage makes no claim the change falsifies.
