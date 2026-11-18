```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the RenderToDisk in-place write diff.

**Cases run**

- `RenderToDisk` with an existing flat output file such as `proposal-a.md`: `os.WriteFile` truncates the file and rewrites it. The inode stays, the content is new and the error is nil. Correct.
- `RenderToDisk` with a new flat output file: the file is created with mode 0600, the same mode `os.CreateTemp` gave the old writes. Correct.
- `RenderToDisk` with an existing file of mode 0644, as a git checkout or sync leaves it: `os.WriteFile` keeps the file's existing mode. Correct.
- `RenderToDisk` with a `relPath` holding a subdirectory, from `readme: docs/Board.md` or `design_prefix: designs/`: `os.WriteFile` returns ENOENT because `docs/` does not exist. The old `AtomicWriteBytes` called `MkdirAll` first. Incorrect: finding 1.
- `RenderToDisk` with `readme: ../Board.md` or an absolute readme: the file is written outside `boardPath`. The old `PathGuard` refused this. Incorrect: finding 1.
- `RenderToDisk` when `os.WriteFile` fails after truncating, for example ENOSPC on file k: file k is left empty or partial. The files before it hold the new render and the files after it hold the old one. The error is returned, so neither the manifest nor the cleanup runs. The next render rebuilds everything. Before the change, file k kept its old content. Finding 2, LOW.
- Error message on a failed write: `write <relPath>: <err>` is wrapped with `%w`, so nothing is swallowed. Correct.
- `board.go:192` when the render fails: `store.Save()` has already run, so the JSON is committed and the caller gets `render: …`. That was the same before the change and is not new.
- `TestRenderToDiskRewritesAnExistingFileInPlace` run against the old `AtomicWrite`: the rename gives a new inode, so `os.SameFile` returns false and the test fails, so it does pin the change. The second-render body assertion is correct. Correct.

**Findings**

1. **MEDIUM · design** · `internal/boardengine/render.go:49`
   Replacing `fsx.AtomicWrite` with `os.WriteFile` dropped two things for every rendered `relPath`: the `PathGuard` check and the creation of the parent directory.
   - A `readme` or `design_prefix` naming a subdirectory, which `template.yaml:1-2` documents as "relative to board dir", now makes every render fail with ENOENT.
   - A `readme` of `../x` or an absolute path now writes outside the board directory.
   - Nothing else checks these values: `config.go:198-210` validates only `design_prefix` against `decision-`, and `readme` is never validated.
   - Slugs are not always validated either: `ValidateSlug` is nil for Boards that make no slug (`config.go:53-56`), and replay and owner renders do not validate.

   Evidence: `internal/fsx/fsx.go:53` (`MkdirAll`), `internal/fsx/fsx.go:80` (`PathGuard`), `internal/boardengine/template.yaml:1`, `internal/boardengine/config.go:198-210`.

   Fix: call `fsx.PathGuard(relPath)` before the write and `os.MkdirAll(filepath.Dir(path), 0o755)` before `os.WriteFile`. Both keep the in-place rewrite.

2. **LOW · design** · `internal/boardengine/render.go:48-51`
   A write that fails after `os.WriteFile` has truncated the file leaves that board file empty or partial until the next successful render. The old temp-and-rename write kept the previous content on failure.

   Evidence: `internal/boardengine/render.go:49`, `internal/fsx/fsx.go:57-75`.

   Fix: accept this, as the summary's design decision does, or say in the doc comment that a failed render can leave a truncated view.

Tool calls:
1. Grepped the yaml files for `readme:` and `design_prefix:` to see whether a configured output name can hold a directory. The manifest held only the struct and not the template's documented meaning.
2. Grepped boardengine for slug validation to see whether a slug can reach a `relPath` unvalidated. The manifest did not cover slug validation.
