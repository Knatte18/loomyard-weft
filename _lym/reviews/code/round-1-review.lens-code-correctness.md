```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the heddle anchor move, with one concrete input run per changed function or branch.

**Cases run**

- `seedWarpExcludeLines` (`internal/fabricengine/warpexclude.go:19-41`), anchorRel `"."`: `path.Join(".", "CLAUDE.local.md")` gives `CLAUDE.local.md`, so the line is `/CLAUDE.local.md`. Correct.
- `seedWarpExcludeLines`, anchorRel `"src/csharp/NORCE.Models"` with listed path `.heddle`: the line is `/src/csharp/NORCE.Models/.heddle`. It has a leading slash, forward slashes and no double separators. Correct.
- `seedWarpExcludeLines`, anchorRel `""`: `path.Join` ignores the empty element. Correct.
- `seedWarpExcludeLines`, line already present or content lacking a trailing newline: the skip and newline handling are unchanged. Correct.
- `Config.WarpExcludePaths` and `SeedWarpExclude` (`warpexclude.go:54-64`): the returned paths stay anchor-relative.
- `Render` (`render.go:99-130`), exclude checks: `Excluded` holds anchor-relative names and is compared against `target.File` and `tempDirName`, which are also anchor-relative. Correct.
- `Render`, `tempDir` at the anchor: the temp file and the target are in the same directory, so the rename cannot cross a filesystem. Correct.
- `Render`, anchor directory missing: `MkdirAll(tempDir)` would create it. The anchor always exists in a wired worktree, so this is benign.
- `fileText` (`render.go:256-268`): the `CLAUDE.md` and `.claude/CLAUDE.md` check and the `AGENTS.md` opening now use the anchor. The `@AGENTS.md` imports resolve beside `CLAUDE.local.md`, which is now at the anchor. Correct.
- `Seam.TookIn` (`seam.go:31-44`), anchor equal to root: `filepath.Rel` gives `"."`, `path.Join(".", rel)` gives `rel`, and the old root-relative matching is preserved. Correct.
- `Seam.TookIn`, anchor `src/app` and `ev.Paths` `["src/app/AGENTS.md"]`: both targets are selected. Correct.
- `Seam.TookIn`, `ev.Paths` `["AGENTS.md"]` above the anchor: nothing is selected, and no lock is taken. Correct.
- `Seam.TookIn`, empty `ev.Paths`: nothing is selected. Correct.
- `Seam.TookIn`, `filepath.Rel` error: it returns an error with context. Reachable only if `Anchor` is not under `Root`, and `EventFor` guarantees it is.
- `Seam.TookIn`, Windows: `ToSlash` is applied before `path.Join`, and git paths use slashes. Correct.
- `EventFor` (`worktreeseam.go:57-64`): `Anchor` is `l.AnchorPath()`, which the cwd-resolution background file says equals `Join(WorktreePath, AnchorRel)`. This is the same `Location` that `SeedWarpExclude` uses for the exclude lines, so the files and their exclude lines agree.
- `lym heddle render <other>` (`internal/heddlecli/verbs.go:331-337`): the event comes from the named worktree's own `Location`, so it carries that worktree's anchor. Correct.
- `lym heddle show` (`verbs.go:295`): it uses the current location's `AnchorPath()`, which matches what a render there would read. Correct.

**Findings**

- **F-A, LOW, design.** `internal/heddleengine/render.go:103-117`: a worktree in an anchored hub that heddle already rendered keeps its old marked `CLAUDE.local.md` or `AGENTS.override.md` at the git root.
  - Render now only touches anchor paths, so the old file is never updated or removed.
  - Claude Code still loads it as an ancestor file, so the same instructions load twice.
  - Evidence: `render.go:103` joins only `req.Anchor`, and the diff adds no cleanup of the root path.
  - The decision record asks for no migration, and the board entry says the hub holds its migration, so this may be accepted. I report it as an edge on the state left behind.

No other defects found in this lens.

**Tool calls:** none. The manifest and the diff already held every definition I needed.
