```yaml
lens: code-tests
outcome: reported
```

## code-tests lens: what pins each changed behaviour

- **Each output file is rewritten in place, keeping its inode** (`render.go:49`): pinned by `TestRenderToDiskRewritesAnExistingFileInPlace` (`internal/boardengine/render_test.go:43`).
  - Its `os.SameFile` check would fail on the old code. `fsx.AtomicWrite` creates a temp file with `os.CreateTemp` while the old file still exists, so the temp file gets a different inode, then renames it over the target (`internal/fsx/fsx.go:57,72`).
  - So the test was red on the old code and is green on the new.
  - Testing only the design file is enough: README and the project files go through the same loop.
- **A re-render writes the new content**: pinned by the same test's `HasSuffix(..., "second body")` check. `TestRenderToDiskManifestCleanup` (`render_test.go:78`) also covers it across consecutive renders.
- **A new file is created on the first render**: covered by the first `RenderToDisk` call in both tests above.
- **A missing parent directory is no longer created, and `PathGuard` no longer runs on output paths**: no test.
  - A grep of `internal/boardengine` found no test with a `Readme` or `DesignPrefix` holding a `/`, and no `PathGuard` assertion. Only `internal/fsx/fsx_test.go:16` tests `PathGuard`, directly.
  - Whether dropping the guard is a defect belongs to the correctness lens. Here the gap is only that nothing pins it.
- **A failed write now leaves a truncated or partial file where the old content used to survive**: no test. `RenderToDisk` has no write-failure test, before or after the diff (grep of `render_test.go` found no `permkit` and no error-expecting call).
- **Doc comments in `doc.go` and on `RenderToDisk`**: static prose, exempt.

The new test meets the repo's test conventions: `tiergate.Require(t, 0)` comes first, then `t.Parallel()`, `t.TempDir()`, and tier0 with no spawn. The redundancy report lists it as unique, and I found no existing test that checks file identity, so it is not redundant.

## Findings

1. **LOW**, class `design`, at `internal/boardengine/render.go:49`
   - No test covers what the diff changed for nested output paths: a `readme` or `design_prefix` holding a subdirectory used to get its directory from `MkdirAll` and its path checked by `PathGuard`.
   - Evidence: the removed `fsx.AtomicWrite` call (diff line 32), `internal/fsx/fsx.go:53,80`, and the grep that found no such test in `internal/boardengine`.
2. **LOW**, class `design`, at `internal/boardengine/render.go:49-51`
   - The failure path's outcome changed and no test covers it: a write that fails now returns `write <relPath>` and leaves the target truncated or partly written, where the old code left the previous content intact.
   - Evidence: `fsx.go:62-75` (the temp file is removed on error and the target is untouched) against `os.WriteFile` truncating first. No failing-write test exists in `render_test.go`.

Tool calls:
1. Grepped `internal/boardengine` and `internal/fsx` tests for nested output paths and `PathGuard` assertions, because the manifest did not list the test bodies beyond lines 1–40.
2. Grepped `render_test.go` for write-failure tests (`permkit`, expected errors), because I had not read the test file past its header.
