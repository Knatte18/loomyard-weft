# Card 1 — pause-record

**Create:**
- `internal/lyxdirs#PauseDirName`
- `internal/pauserecord/doc.go`
- `internal/pauserecord#RecordFileName`
- `internal/pauserecord#ScopeHub`
- `internal/pauserecord#Retention`
- `internal/pauserecord#ResumeLine`
- `internal/pauserecord#Geometry`
- `internal/pauserecord#Session`
- `internal/pauserecord#Entry`
- `internal/pauserecord#Record`
- `internal/pauserecord#Interval`
- `internal/pauserecord#Coverage`
- `internal/pauserecord#Read`
- `internal/pauserecord#Update`
- `internal/pauserecord#Record.Covering`
- `internal/pauserecord#Record.Owner`
- `internal/pauserecord#Record.CoverageOf`
- `internal/pauserecord#ReadCoverage`
- `internal/pauserecord#Excluded`
- `internal/pauserecord#CoveredError`
- `internal/pauserecord/record_test.go`
- `internal/pauserecord/leaf_enforcement_test.go`
- `internal/pauserecord/testmain_test.go`

**Edit:**
- `internal/lyxdirs/dirs.go`
- `pattern/PATTERN-leaf-packages.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the pause record (D2): one JSON file, written under a lock beside it, that every reader of a pause consults.
`lyxdirs.PauseDirName` is the directory under the hub's board-level `.lyx` that holds it; `dirs.go`'s file comment stops calling the two literals the package's only exported symbols.
`internal/pauserecord` is a leaf importing the standard library and `internal/lock` only.

Types, in `record.go`:
- `Geometry` is the told pair every reader is handed: `RecordPath` (the record file) and `Slug` (the reading process's own pair slug, empty in the prime and in standalone mode). Only `internal/hubgeom` and `internal/standalonegeom` construct it (card 4).
- `Session` is one session an entry owns: `Strand` (reed strand name), `GUID`, `Slug` (empty for the orch), `Orch bool`, and `ResumeNoticed bool`, which the orch watcher sets once it queued the orch's resume line (card 21).
- `Entry` is `Scope` (`ScopeHub` or a slug), `PausedAt`, `GraceEnd`, `Caller` (the pausing strand name or `EXTERNAL`), `Sessions []Session` and `ResumedAt` (zero while live). JSON keys are snake_case (`paused_at`, `grace_end`, `resumed_at`).
- `Record` is `Live []Entry` and `Resumed []Entry`; at most one live entry per scope.
- `Interval` is `Start`, `End`; `Coverage` is `Covered bool`, `Entry Entry` (the covering live entry, zero when uncovered) and `Intervals []Interval`.

Functions:
- `Read` returns the empty record for a missing file and an error naming the path for an unreadable or undecodable one.
- `Update` takes the write lock at `path + ".lock"` (`lock.AcquireWriteLock`, creating the directory first), reads, calls `mutate`, drops every resumed entry whose `ResumedAt` is older than `Retention` before `now`, and writes atomically (temp file in the same directory, then rename); a `mutate` error writes nothing and is returned.
- `Record.Covering` returns the oldest live entry whose scope is `ScopeHub` or `slug`; an empty slug is covered by a hub entry only.
- `Record.Owner` returns the live entry whose `Sessions` holds `guid`.
- `Record.CoverageOf` returns the covering entry and every interval of an entry covering `slug`: a live entry's interval runs from `PausedAt` to `now`, a resumed one's to `ResumedAt`.
- `ReadCoverage` is `Read` then `CoverageOf`.
- `Excluded` returns the length of the union of `intervals` clipped to `[from, to]`, so overlapping pauses and sleeps are subtracted once.
- `CoveredError` renders the refusal every held entry point returns (cards 13 and 15): `pause: <a hub-wide pause | the pause of <slug>> has held this run since <paused_at>; way forward: run "lyx resume" | "lyx resume <slug>", then <retry>`, naming `lyx resume` for a hub entry and `lyx resume <slug>` for a slug entry.
- `ResumeLine` is D6's fixed resume line, read by the resume sequence (card 22) and the orch watcher (card 21).

Tests, in `record_test.go` (package `pauserecord`), one table test per function: `Read` of a missing, valid and undecodable file; `Update` creating the file, rejecting on a `mutate` error with the file unchanged, and pruning a resumed entry older than `Retention` while keeping a younger one; two goroutines updating the same record each append their entry and the file decodes with both; `Covering` with a hub entry, a slug entry, both (oldest wins) and none, and an empty slug under a slug entry; `CoverageOf` intervals for a live and a resumed entry; `Excluded` with disjoint, overlapping and partly outside intervals; `CoveredError` text for both scopes.
`leaf_enforcement_test.go` calls `scankit.AssertImportAllowlist(t, "internal/pauserecord", "github.com/Knatte18/loomyard/internal/lock")`.
`testmain_test.go` runs the binary under `tmuxkit.Main` from the external package `pauserecord_test`, as `internal/lyxtemp` does, because `internal/gitexec` imports this package from card 5 on.
`pattern/PATTERN-leaf-packages.md` gains the `internal/pauserecord` section (standard library and `internal/lock` only), and `cmd/lyx/testdata/test-budget.yaml` gains its row.

**ImpactSummary:** New leaf package and one new lyxdirs constant; no existing code calls either yet.
