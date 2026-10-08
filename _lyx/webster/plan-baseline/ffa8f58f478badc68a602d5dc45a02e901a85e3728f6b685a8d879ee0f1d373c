# Card 3 — Reconcile failures name their file

**Create:**
- `internal/configsync#FileError`
- `internal/configsync#FileError.Error`
- `internal/configsync#FileError.Unwrap`

**Edit:**
- `internal/configsync#ReconcileAll`
- `internal/configsync#ReconcileHubWideAt`
- `internal/configsync#retireHubWideCopy`
- `internal/configsync#decodeLeafValues` -> `func decodeLeafValues(data []byte) (map[string]any, error)`
- `internal/configsync/configsync_test.go`

**Intent:**
Decision 9 needs a failed reconcile to name the file that failed; today `ReconcileAll` and `ReconcileHubWideAt` return errors that name only the module, and return no results, so a caller cannot tell which file to fix.

`FileError` carries `Module string`, `Path string` (the absolute path of the config file that failed) and `Err error`.
`Error()` renders `<Path>: <Err>`, so the existing message (`reconcile batcher: …`, `read config for fabric: …`) survives as the suffix; `Unwrap` returns `Err`.
Every failure `ReconcileAll` and `ReconcileHubWideAt` return for one module is wrapped in a `*FileError` naming that module and the file the failure is about: the module's config file for a read, migrate, reconcile or write failure, a removed legacy file's path for its removal failure.
`retireHubWideCopy` wraps its failures the same way, naming the per-worktree copy or the hub file, whichever failed to read or parse, and the copy for a failed removal.
`decodeLeafValues` drops its `path` parameter and stops prefixing its parse error with the path (`parse: %w`), since the `FileError` around it now names the file; its two calls in `retireHubWideCopy` drop the argument.
The `configsync.go` file header gains one sentence saying failures are `*FileError` values naming the file.

No test asserts the old texts (grep `read config for`, `reconcile %s`, `write config for`, `migrate %s`, `parse %s` finds only `configsync.go`), and `lyx config reconcile`'s and `lyx fabric reconcile`'s error envelopes gain the path prefix.

Tests in `configsync_test.go`: a new table test whose rows each make one file unparseable (a per-worktree `batcher.yaml` for `ReconcileAll`, the hub `fabric.yaml` for `ReconcileHubWideAt`, and a per-worktree copy of a hub-wide module beside a valid hub file for `ReconcileAll`'s retire path) and assert `errors.As` yields a `*FileError` whose `Module` and `Path` name that file and whose message starts with the path.

**ImpactSummary:** Error shape only: configsync's two reconcile entry points and their two internal helpers wrap failures in a typed error; callers printing `err.Error()` gain a path prefix.

**Commit:** 3: configsync: a reconcile failure names the file that failed
