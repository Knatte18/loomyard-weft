```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each function, branch and error path the diff changes against a concrete input.

**List: function or branch, the case I ran, and the result**

1. **`RenderToDisk` write loop, existing output (`proposal-a.md` already on disk, new body).** `os.WriteFile` opens with `O_TRUNC`, writes the new content and keeps the inode and the existing mode. Correct.
2. **`RenderToDisk` write loop, new output (first render into an empty `boardPath`).** `O_CREATE` with 0o600, the same mode `os.CreateTemp` gave before. Correct.
3. **`RenderToDisk` write loop, `out.Readme = "wiki/Home.md"`, with no `wiki/` directory.** Before the diff, `AtomicWriteBytes` ran `MkdirAll(dir)`. Now `os.WriteFile` returns ENOENT, so `RenderToDisk` returns `write wiki/Home.md: …`. In `board.go:180-194` this happens after `store.Save()`, so every board write reports a render error and the board files never render. **Not correct (finding 1).**
4. **`RenderToDisk` write loop, `out.Readme = "../outside.md"`.** This value can come from `board.yaml` or `LYM_README`, via `template.yaml:1`. Before, `PathGuard` refused it with "parent directory reference not allowed". Now `filepath.Join(boardPath, "../outside.md")` resolves to the board directory's parent, and the render writes, and on later renders truncates, a file outside `boardPath`. The manifest then records the name, and a later render's cleanup `os.Remove(filepath.Join(boardPath, name))` deletes that outside file. A `design_prefix` holding `../` takes the same path; `checkDesignPrefix` only checks the `decision-` clash. **Not correct (finding 2).**
5. **`RenderToDisk` write loop, `out.Readme = ""`.** Before: `PathGuard` returned "empty path". Now `os.WriteFile(boardPath)` fails with EISDIR. Both return an error, so this is correct, with a different message.
6. **`RenderToDisk` write loop, the write fails mid-file (ENOSPC or EIO on the second file).** Before, the temp file was removed and the old content stayed whole. Now the target is truncated or partial. `RenderToDisk` returns at once, so the manifest is not rewritten and the remaining files keep their old content. The board stays consistent: `board.json` is saved and the next render rebuilds every file. But a partial rendered `.md` now sits in the Bolt worktree. The next `lym board sync` `commitDirty` (`sync.go:60-74`) stages and commits it, and only a later successful render repairs it. The error is reported, not swallowed. **Not correct, at low severity (finding 3).**
7. **Concurrent reader during the write.** All three callers render under the board write lock, which `commitDirty` also takes (`board.go` `boardCriticalSection` at lines 97-105, the comment at `replay.go:460`, the comment at `ownerset.go:226`). So git never stages a mid-write file. Only out-of-process readers such as an editor can see a partial file, and the next change event corrects it. Correct.
8. **Error wrapping.** `fmt.Errorf("write %s: %w", relPath, err)`. The `*PathError` from `os.WriteFile` already holds the absolute path, so the message names the path twice. That is harmless. Correct.
9. **`TestRenderToDiskRewritesAnExistingFileInPlace`.** On the old code, `os.CreateTemp` plus `os.Rename` gives a new inode, so `os.SameFile` is false and the test fails. On the new code it passes. `renderDesigns` builds the design file as header + `"\n"` + `e.Body`, and the body `"second body"` has no trailing newline, so `HasSuffix` holds. Correct.

**Findings**

1. **MEDIUM, design.** `internal/boardengine/render.go:49`
   - Summary: the swap from `fsx.AtomicWrite` drops the `MkdirAll` of each output's parent directory. A configured `readme` holding a subdirectory, such as `wiki/Home.md`, now fails every render with ENOENT, where it used to create the directory.
   - Evidence: `internal/fsx/fsx.go:53` (the `MkdirAll`), `render.go:49`, `template.yaml:1` ("relative to board dir").
   - Fix: restore the parent-directory creation, or refuse a nested `readme` at config load.
2. **MEDIUM, design.** `internal/boardengine/render.go:49`
   - Summary: the swap also drops `fsx.PathGuard`, the only check on the config-supplied `readme` and `design_prefix` output paths. A value holding `..` now makes the render write and truncate a file outside `boardPath`, and makes the manifest cleanup delete it later.
   - Evidence: `internal/fsx/fsx.go:79-84`, `render.go:49` and `render.go:54-59`, `config.go:205-213` (`checkDesignPrefix` checks only the `decision-` prefix).
   - Fix: keep `fsx.PathGuard(relPath)` before the in-place write.
3. **LOW, design.** `internal/boardengine/render.go:49`
   - Summary: when a write fails after `O_TRUNC`, the rendered file is left empty or partial, where the temp-and-rename path left the old content whole. A later `lym board sync` (`commitDirty`) commits and pushes that truncated file until a later render succeeds.
   - Evidence: `render.go:48-51`, `sync.go:60-74`, `board.go:190-194` (the render error is returned after `store.Save()`).
   - Fix: none is required for the fix's purpose. Accept it, and state it in the `RenderToDisk` doc's failure contract, or write in place only once the content is fully prepared, which `os.WriteFile` already does except on an I/O failure.

Tool calls: none. My context already held every definition and call site this lens needed.
