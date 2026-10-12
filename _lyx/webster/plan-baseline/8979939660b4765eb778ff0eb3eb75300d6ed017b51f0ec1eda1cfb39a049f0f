# Card 15 — fsx-newest-modtime

**Create:**
- `internal/fsx/newest.go`
- `internal/fsx#NewestModTime`
- `internal/fsx/newest_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add `fsx.NewestModTime(path)`, the age both the batten orphan-scratch rule (one day) and the `.scratch/` segment (14 days) read: the newest modification time of `path` and of everything under it.
It `os.Lstat`s `path`; a file or a link answers its own modification time, a link never followed;
a directory is walked with `filepath.WalkDir`, which follows no link, and answers the newest `Lstat` modification time among the directory itself and every entry below it.
An error from the walk is returned; a not-exist on `path` itself is returned as is, so a caller can tell an absent directory with `errors.Is(err, fs.ErrNotExist)`.
`newest_test.go`, untagged and parallel, sets modification times with `os.Chtimes` and covers a file, an empty directory, a deep file newer than its directories, a link to a directory holding a file dated an hour ahead, where the answer is the link's own time and never the target's, and a missing path.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/fsx` row grows by the new top-level test.

**ImpactSummary:** One new exported helper in fsx and its budget row; no existing fsx function changes.
