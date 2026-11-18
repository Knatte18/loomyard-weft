```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each behaviour the diff changes, I looked for the callers, docs, tests and rules that depend on it.

**Per change, the set found and whether it is covered:**

1. **`RenderToDisk` writes each output with `os.WriteFile` instead of `fsx.AtomicWrite`.** The old call also ran `PathGuard` and `MkdirAll`.
   - Callers, signature unchanged:
     - `board.go:192`
     - `replay.go:613`
     - `ownerset.go:336`, through `renderOwnerWrite`, which is reached from `board.go:241`, `ownerset.go:314`, `registrysetter.go:148` and `holdwrite.go:94`
     - Covered: none needs a change.
   - Config inputs that become output paths:
     - `readme` (`template.yaml:1`, "relative to board dir")
     - `design_prefix` (`config.go:198-213`, checked only against the `decision-` name)
     - Not covered: neither key is checked for a subdirectory, an absolute path or `..`. The old write guarded these paths and created the parent directory; the new one does neither.
   - `replay.go:842` writes origin's files with `AtomicWrite`, and `RenderToDisk` then rewrites the same paths in place. Covered: no change needed.
   - `render.go:1-14` file header, "RenderToDisk writes the files": still true. Covered.
   - `docs/research/loom/lym-state-map-2026-11-15.md:80` lists only the rendered files and their writer, not the write method. Covered.
   - PATTERN entries and `pattern/` background files: none describes how board render output is written. This round declares no PATTERN changes. Covered.
   - Tests: none in boardengine asserts a temp-file name, `PathGuard` on render outputs or a file mode. Covered.
2. **The `RenderToDisk` doc comment and the `doc.go` paragraph state the in-place write.** No other doc in the package states the old atomic write: a grep of `doc.go` for atomic, temp file or rename found nothing else. Covered.
3. **Commit messages of the range (`28c270f4`, `b05e3b1d`).** No `Pattern-Departure:` or `Loosened-Check:` trailer is needed, since no test check is loosened and no PATTERN way is departed from. Covered.

**Findings:**

- **MEDIUM, design.** `internal/boardengine/render.go:49`
  - Summary: dropping `fsx.AtomicWrite` also drops `PathGuard` and `MkdirAll` for render output paths. These paths come from the `readme` and `design_prefix` config keys, which no check limits to a plain file name. A `readme` such as `docs/Board.md` or a `design_prefix` such as `designs/` used to work, because the parent directory was created; every board write now fails with "no such file or directory". An absolute or `../` `readme` now writes outside the board directory instead of being refused.
  - Evidence: `internal/fsx/fsx.go:53` (`MkdirAll`) and `fsx.go:79-84` (`PathGuard`); `internal/boardengine/template.yaml:1` (`readme` is "relative to board dir"); `internal/boardengine/config.go:198-213` (`checkDesignPrefix` checks only the `decision-` clash).
  - Fix: put back the parent-directory creation and the path guard before `os.WriteFile`, for example `fsx.PathGuard(relPath)` plus `os.MkdirAll(filepath.Dir(...))`; or validate `readme` and `design_prefix` in `LoadConfig`.

**Tool calls:**

1. Grepped `doc.go`, `docs`, `pattern` and `PATTERN.md` for atomic, temp or rename wording on render output. The manifest held no doc search beyond the diff.
2. Grepped boardengine tests for `PathGuard`, mode and temp-file asserts, and all `RenderToDisk` and `renderOwnerWrite` callers. The manifest lacked the callers one hop beyond `renderOwnerWrite`.
3. Grepped the state-map doc and the subdirectory config wording. These were hits from call 1 that the manifest had not read.
