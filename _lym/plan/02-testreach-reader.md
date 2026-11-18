# Card 2 — testreach-reader

**Create:**
- `internal/testreach#Reader`
- `internal/testreach#Reach`
- `internal/testreach#MapSource`
- `internal/testreach#Open`
- `internal/testreach#Reader.TestsReaching`
- `internal/testreach#Reader.Maps`
- `internal/testreach#Reader.SpawnMarked`
- `internal/testreach#Reader.StaleLinkers`
- `internal/testreach#Fresh`
- `internal/testreach#ReadPackageMap`
- `internal/testreach_test#TestReader`
- `internal/testreach_test#TestFresh`
- `internal/testreach_test#TestReaderIndex`

**Extends:**
- `internal/testreach/doc.go`

**Uses:**
- `internal/testcover#ReadMap`
- `internal/testcover#WriteMap`
- `internal/testcover#Map`
- `internal/testcover#MapTest.Measured`
- `internal/testcover#ContentHash`
- `internal/state#ReadJSON`

**Intent:**
Add D1's reader of the coverage maps across packages, reading its index when one is current; card 6 adds the index's one writer.

`Open(module, sharedDir, worktreeDir)` reads two map directories, each a `testcover.Dir`:
the hub's shared maps (`sharedDir`, under the board directory) and the worktree's own maps (`worktreeDir`, under its anchor; "" in the prime, which keeps none).
An absent `sharedDir` or `worktreeDir` reads as a directory with no maps and refuses nothing, as an absent cache does under `PATTERN-lym-files`:
a task worktree with no anchor maps and a hub whose `--map` never ran are common.
It walks each directory for `map.json` files (the package is the file's directory relative to the maps directory) and reads them through `testcover.ReadMap`.
- For a package both directories hold, the worktree's map wins when `Fresh` holds for it; otherwise the map whose `BuiltAt` is later wins.
- A map holding only a `BuildFailure`, or one naming a block or function index outside its tables, is passed over as if absent.
- A map file that does not parse is passed over too, with a Warn naming its package and the way forward, rebuild it with `lym gate test --map` from the prime;
  so one corrupt map never takes the reader down for every other package.
- The worktree's own maps are few and are read whole, never indexed.
- The shared maps' reverse table, from function to reaching tests, comes from the index file beside them when its stamp equals the directory's current stamp.
  The stamp is the set of map files in the directory, each with its size and modification time, so an added or removed map file is a stamp difference; the index file name is one named constant.
  An absent index, an unreadable one, or one whose stamp differs is never refused: `Open` builds the same table in memory from the maps and goes on.
  It never writes the index.
  The table keeps, per entry, the package whose map it came from, so a package whose worktree map wins drops its shared entries and takes the worktree map's.

Reach rules (D1):
- In its own package a test reaches a function when its `MapTest.Blocks` hold a block whose `File` and `Func` are the function's.
- In another package a test reaches a function when its `MapTest.Funcs` hold the function's `FuncKey`.
- An entry that is not `Measured` (an outcome or a spawn mark) reaches nothing.

`TestsReaching(fn)` returns every test that reaches `fn` as a `Reach`: `Package` (the test's import path), `Test`, `Own` (the test is of `fn`'s package),
`Funcs` (how many functions its entry reaches, own and linked together) and `MapCommit` (the commit of the map it came from).
They come own package first, then other packages; within each group fewest `Funcs` first, then package and name.
`Maps()` lists the `MapSource` of each map the reader read: `Package` (module-relative), `Commit`, `BuiltAt` and `Worktree`, set for a worktree map.
`SpawnMarked(pkg)` reports whether the package's map holds an entry whose spawn verdict marks it out of process or unresolved (D4's caveat).
`StaleLinkers(pkg)` names, sorted, the module-relative packages whose maps record `pkg` (an import path) under `Linked` with hashes that differ from the worktree's files of `pkg` (D4's caveat for a new function).

`Fresh(module, pkg, m)` reports whether `m`'s recorded hashes match the worktree's files under `module.Dir`:
its `Files` and `OtherFiles` against the package's files, and each `Linked` package's hashes against that package's files, each by `testcover.ContentHash`.
A file on one side alone makes it stale. It reads the disk only.

`ReadPackageMap(dir, pkg)` reads one package's map from one maps directory by `testcover.ReadMap`, with `Open`'s pass-over rules:
a map holding only a `BuildFailure` or naming an index outside its tables comes back not found;
a map that does not parse is an error naming the package and the way forward, rebuild it with `lym gate test --map`.
It is the read every new gatecli caller of one package's map uses (cards 5 and 13), so card 3's guard admits no new direct `testcover.ReadMap` caller.

Tests, all tier0 over map directories a test writes with `testcover.WriteMap` under `t.TempDir()` and package files it writes beside them:
- `TestReader` is one table over `Open` and the lookups:
  a function reached by a test of its own package and by one of another package comes back with both, the own package's first and the fewer-functions test first within a group;
  an outcome entry and a spawn-marked entry reach nothing, and `SpawnMarked` reports the latter;
  a worktree map whose hashes match the worktree overrides the shared map of its package, and with a stale worktree map the later-built map is read;
  a missing worktree maps directory still answers from the shared maps, and a missing shared directory answers with no reach;
  `StaleLinkers` names a package whose map links `pkg` with older hashes and leaves out one whose hashes match;
  a map holding only a `BuildFailure`, a map naming a block index outside its table and a map file that does not parse are each passed over while the other packages still answer;
  `ReadPackageMap` returns a readable map, not found for the `BuildFailure` map, and an error naming the package for the unparsable one.
- `TestFresh` is one table: matching files, a changed own file, an added file, a changed `OtherFiles` entry and a changed linked package's file.
- `TestReaderIndex` is one table over the index states this card can produce: no index, and an index file that does not parse.
  The unparsable row's `TestsReaching` answers equal those of the no-index row, and `Open` leaves the index file's bytes as it found them.

`doc.go` gains the reader's rules and its read of the index.

**ImpactSummary:** Adds the reader beside card 1's finder inside the new package; only its doc.go changes, and nothing outside the package calls it yet.
