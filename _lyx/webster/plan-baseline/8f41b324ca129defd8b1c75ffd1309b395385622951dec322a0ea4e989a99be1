# Card 1 — fswatch-leaf

**Create:**
- `internal/fswatch#Watcher`
- `internal/fswatch#Event`
- `internal/fswatch#Watch`
- `internal/fswatch#Watcher.Events`
- `internal/fswatch#Watcher.Close`
- `internal/fswatch/doc.go`
- `internal/fswatch/fswatch_test.go`
- `internal/fswatch/leaf_enforcement_test.go`

**Edit:**
- `go.mod`
- `go.sum`
- `PATTERN.md`
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`
- `docs/shared-libs/README.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Add `internal/fswatch`, the one file-change event source for every lyx process (D3), over `github.com/fsnotify/fsnotify` pinned at v1.9.0, which is in the module cache already: add the direct requirement to `go.mod` and its sums to `go.sum`.
The package imports only the standard library, fsnotify and `internal/logger`, and `leaf_enforcement_test.go` pins that set through `scankit.AssertImportAllowlist`, in the manner of `internal/pattern/leaf_enforcement_test.go`.
`Watch(dir, names...)` opens one fsnotify watcher on `dir` and returns a `Watcher` whose `Events()` channel delivers one `Event` per create, write, rename or remove of an entry whose base name is in `names`, or of any entry when `names` is empty; `Event` carries the entry's base name and the operation as a string.
It coalesces nothing and debounces nothing: the caller owns timing.
An fsnotify failure at open or add (the inotify watch limit, an unsupported filesystem) is returned as an error from `Watch`, so the caller falls back to its poll cadence.
fsnotify's own error channel is drained and logged at Debug.
`Close` stops the watcher and closes the `Events()` channel.
`fswatch_test.go` is untagged and spawns nothing: it creates, writes and removes files in `t.TempDir()` and asserts that an event arrives for each of a create, a write and a remove of a named entry, that an unnamed entry yields none, that `Close` ends the channel, and that an unwatchable directory (a missing path) returns an error.
`doc.go` holds the package doc: the surface, the no-coalescing rule, and the fallback contract.
`PATTERN.md`'s `PATTERN-leaf-packages` line adds `fswatch` to its package list, and `pattern/PATTERN-leaf-packages.md` gains an `internal/fswatch` section stating the closed import set (standard library, `github.com/fsnotify/fsnotify`, `internal/logger`).
`docs/overview.md`'s module tree gains an `internal/fswatch/` line beside the other leaves, and `docs/shared-libs/README.md`'s implementation-only list gains an `internal/fswatch` entry.
`cmd/lyx/testdata/test-budget.yaml` gains the `internal/fswatch` row.

**ImpactSummary:** One new direct dependency and one new leaf package; the module file, the leaf PATTERN entry and the module docs change, and no existing code is touched.
