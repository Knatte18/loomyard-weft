# Card 17 — Pair teardown archives the pair's logs and prunes the archive

**Create:**
- `internal/pairteardown/archive.go`
- `internal/pairteardown#logArchiveRetention`
- `internal/pairteardown#logArchive`
- `internal/pairteardown#logArchive.write`
- `internal/pairteardown#logArchive.prune`
- `internal/pairteardown#archiveSource`
- `internal/pairteardown#Teardown.archivePairLogs`
- `internal/pairteardown/archive_test.go`
- `internal/pairteardown#TestLogArchive`

**Edit:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/pairteardown/doc.go`
- `pattern/PATTERN-pair-teardown.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/fabricengine#LogArchiveDir`
- `internal/logger#LogsDirAt`
- `internal/verifytree#Dir`
- `internal/verifytree#IsLogPath`

**Intent:**
D14 and D15: the one route every pair teardown takes (`PATTERN-pair-teardown`) copies the pair's trace logs and verify logs into a git-ignored hub archive before removal, and keeps that archive to 30 days.

`Teardown` gains an `archive func(slug string)` substrate, set by `New` to `archivePairLogs` and by the fakes to one that records an `archive` call.
`RemovePair` calls it right before `t.remove`, so on both sequencers (`Run`, and batten's two-row producer, which calls `EndSession` then `RemovePair`) it runs after the session end and the run-lock wait, when no step writes into the pair any more, and before `Topology.Remove`.
It never fails or delays the teardown: it returns nothing, and `RemovePair` goes on to the removal whatever it did.
It copies and never moves, so a removal the topology then refuses leaves the pair's logs whole, and the next teardown writes a fresh entry.

`archivePairLogs` resolves the archive through `logArchive{root: fabricengine.LogArchiveDir(t.prime.HubPath), now: time.Now, removeAll: os.RemoveAll}` and two sources under the task worktree's anchor (`t.taskAnchor(slug)`):
`trace`, every regular file directly in `logger.LogsDirAt(anchor)`;
`verify`, every file in `verifytree.Dir(anchor)` that `verifytree.IsLogPath` accepts for `verifytree.NewPaths` over that directory, the pair's `verify-<n>.log` files.
It calls `prune`, then `write`, then, when `write` made an entry, logs one Info line `pairteardown: archived the pair's logs` with the entry path and file count; `write`'s error logs Warn and nothing else.

`write` collects both sources' files first and, when there are none (a pair with neither source, or a task worktree already gone), creates no entry and returns an empty `entry`.
Otherwise it creates `<root>/<slug>/` and then the entry directory `<UTC timestamp>`, in the layout `20060102T150405Z`, exclusively with `os.Mkdir`;
on a name collision it tries `<timestamp>-2`, `-3` and on, so it never writes into an existing entry and two archives of one slug in the same second land in two entries.
It copies each source's files into `<entry>/trace/` or `<entry>/verify/`, keeping their names.
A copy failure removes the step's own partial entry with `os.RemoveAll` on that entry alone and returns the error.

`prune` reads the root's slug directories and, inside each, removes every directory whose name parses (with or without a collision suffix) as an archive timestamp older than `logArchiveRetention` before `now`, then removes the slug directory if it is left empty.
It removes through the `removeAll` field, `os.RemoveAll` in production.
An entry whose name does not parse, and any plain file, is left alone; an entry or slug directory already gone, a removal or read error that `errors.Is` `fs.ErrNotExist`, counts as removed with no log; any other failure logs Warn and pruning goes on.
Pruning is outside `package fabricengine`, so `PATTERN-fabric-destruction-chokepoint` does not apply, and the retention is a Go constant, not a config key.

`doc.go` gains the archive step in its sequence and its bounds; `pattern/PATTERN-pair-teardown.md` gains a bullet that the composite copies the pair's trace and verify logs into the hub archive after the run-lock wait and before the removal, never failing the teardown.

Tests:
`TestTeardown_CallSequence`'s rows that reach the removal want `archive` immediately before `remove` (the removal-failure row included, since the archive copies before a refused removal), and the rows that stop before `RemovePair` want no `archive`.
`TestLogArchive`, in the new `archive_test.go`, is table-driven over temp directories with an injected `now`, one row per behavior:
a pair with trace and verify logs gets one entry with both subdirectories and the right files, and the sources are still in place afterwards;
a pair with neither source creates no entry and no slug directory;
a copy failure (a source file made unreadable, skipped where the test runs as root) leaves no entry under the slug, and `write` returns the error;
two writes of one slug at the same `now` land in two distinct entries;
a prune removes an entry 31 days old and the slug directory it empties, keeps an entry 29 days old, and leaves an entry named `not-a-stamp` and a plain file alone;
a prune whose `removeAll` answers an error wrapping `fs.ErrNotExist` for an old entry, as when another teardown removed it first, logs no Warn (captured at Warn), while one whose `removeAll` answers another error logs one.
Two rows drive `archivePairLogs` itself on a `Teardown` whose `prime` is a `lyxcwd.Location` over a temp hub (`AnchorRel` "."), with the pair's logs written under `t.taskAnchor(slug)`:
a pair with logs leaves one entry and logs one `level=INFO` `archived the pair's logs` line carrying the entry path and the file count;
a pair whose trace file is unreadable leaves no entry and logs one `level=WARN` line.
`archive` returns nothing and `RemovePair` calls `t.remove` after it unconditionally, so a failed archive cannot change the teardown's result; `TestTeardown_CallSequence`'s removal rows pin that the removal still follows it.
The rows that capture the process-global logger (the two prune-logging rows and the two `archivePairLogs` rows) run serially; the others call `t.Parallel`.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/pairteardown` row rises from 6 to 7.

**ImpactSummary:** a new best-effort step inside `RemovePair`, the one removal route, plus its substrate field; no result, error or ordering of the existing teardown steps changes, and the archive writes only under `<hub>/_board/.lyx/log-archive`.

**Commit:** 17: pairteardown: archive the pair's logs before removal, keep 30 days
