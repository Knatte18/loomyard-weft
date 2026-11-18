# Card 2 — testreach-reader

**Create:**
- `plan:internal/testreach#Reader` -> `type Reader struct`
- `plan:internal/testreach#Reach` -> `type Reach struct`
- `plan:internal/testreach#MapSource` -> `type MapSource struct`
- `plan:internal/testreach#Open` -> `func Open(module Module, sharedDir, worktreeDir string) (*Reader, error)`
- `plan:internal/testreach#Reader.TestsReaching` -> `func (r *Reader) TestsReaching(fn testcover.FuncKey) []Reach`
- `plan:internal/testreach#Reader.FuncsReachedBy` -> `func (r *Reader) FuncsReachedBy(pkg, test string, among []testcover.FuncKey) ([]testcover.FuncKey, MapSource, bool)`
- `plan:internal/testreach#Reader.Maps` -> `func (r *Reader) Maps() []MapSource`
- `plan:internal/testreach#Reader.SpawnMarked` -> `func (r *Reader) SpawnMarked(pkg string) bool`
- `plan:internal/testreach#Reader.StaleLinkers` -> `func (r *Reader) StaleLinkers(pkg string) ([]string, error)`
- `plan:internal/testreach#Fresh` -> `func Fresh(module Module, pkg string, m testcover.Map) (bool, error)`
- `plan:internal/testreach#ReadPackageMap` -> `func ReadPackageMap(dir, pkg string) (testcover.Map, bool, error)`
- `plan:internal/testreach#WriteIndex` -> `func WriteIndex(module Module, sharedDir string) error`
- `plan:internal/testreach#TestReader` -> `func TestReader(t *testing.T)`
- `plan:internal/testreach#TestFresh` -> `func TestFresh(t *testing.T)`
- `plan:internal/testreach#TestReaderIndex` -> `func TestReaderIndex(t *testing.T)`

**Extends:**
- `internal/testreach/doc.go`

**Uses:**
- `internal/testcover#ReadMap`
- `internal/testcover#WriteMap`
- `internal/testcover#Map`
- `internal/testcover#MapTest.Measured`
- `internal/testcover#ContentHash`
- `internal/state#ReadJSON`
- `internal/state#WriteJSON`

**Intent:**
Add D1's reader of the coverage maps across packages, with its index.

`Open(module, sharedDir, worktreeDir)` reads two map directories, each a `testcover.Dir`:
the hub's shared maps (`sharedDir`, under the board directory) and the worktree's own maps (`worktreeDir`, under its anchor; "" in the prime, which keeps none).
It walks each directory for `map.json` files (the package is the file's directory relative to the maps directory) and reads them through `testcover.ReadMap`.
- For a package both directories hold, the worktree's map wins when `Fresh` holds for it; otherwise the map whose `BuiltAt` is later wins.
- A map holding only a `BuildFailure`, or one naming a block or function index outside its tables, is passed over as if absent.
- A map file that does not parse is passed over too, with a Warn naming its package and the way forward, rebuild it with `lym gate test --map` from the prime;
  so one corrupt map never takes the reader down for every other package.
- The worktree's own maps are few and are read whole, never indexed.
- The shared maps' reverse table, from function to reaching tests, comes from the index file beside them when its stamp equals the directory's current stamp.
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
`FuncsReachedBy(pkg, test, among)` returns the functions of `among` the test reaches, the `MapSource` of the map holding it, and false when no map holds a measured entry of that name.
`Maps()` lists the `MapSource` of each map the reader read: `Package` (module-relative), `Commit`, `BuiltAt` and `Worktree`, set for a worktree map.
`SpawnMarked(pkg)` reports whether the package's map holds an entry whose spawn verdict marks it out of process or unresolved (D4's caveat).
`StaleLinkers(pkg)` names, sorted, the module-relative packages whose maps record `pkg` (an import path) under `Linked` with hashes that differ from the worktree's files of `pkg` (D4's caveat for a new function).

`Fresh(module, pkg, m)` reports whether `m`'s recorded hashes match the worktree's files under `module.Dir`:
its `Files` and `OtherFiles` against the package's files, and each `Linked` package's hashes against that package's files, each by `testcover.ContentHash`.
A file on one side alone makes it stale. It reads the disk only.

`ReadPackageMap(dir, pkg)` reads one package's map from one maps directory by `testcover.ReadMap`, with `Open`'s pass-over rules:
a map holding only a `BuildFailure` or naming an index outside its tables comes back not found;
a map that does not parse is an error naming the package and the way forward, rebuild it with `lym gate test --map`.
It is the read every new gatecli caller of one package's map uses (cards 6 and 14), so card 4's guard admits no new direct `testcover.ReadMap` caller.

`WriteIndex(module, sharedDir)` builds the reverse table from the shared maps and writes it, with its stamp, through `internal/state` to the index file beside the maps.
The stamp is the set of map files in the directory, each with its size and modification time, so an added or removed map file is a stamp difference.
The index file name is one named constant.
Bound: a map rewritten at the same size within one modification-time tick keeps its stamp, so the index can serve its old pairs until the next map run writes it; every answer is advisory.
Card 3 makes the prime's map run its only caller, and card 4 pins that.

Tests, all tier0 over map directories a test writes with `testcover.WriteMap` under `t.TempDir()` and package files it writes beside them:
- `TestReader` is one table over `Open` and the lookups:
  a function reached by a test of its own package and by one of another package comes back with both, the own package's first and the fewer-functions test first within a group;
  an outcome entry and a spawn-marked entry reach nothing, and `SpawnMarked` reports the latter;
  a worktree map whose hashes match the worktree overrides the shared map of its package, and with a stale worktree map the later-built map is read;
  `FuncsReachedBy` returns the reached subset with the map's commit, and false for a test no map holds;
  `StaleLinkers` names a package whose map links `pkg` with older hashes and leaves out one whose hashes match;
  a map holding only a `BuildFailure`, a map naming a block index outside its table and a map file that does not parse are each passed over while the other packages still answer;
  `ReadPackageMap` returns a readable map, not found for the `BuildFailure` map, and an error naming the package for the unparsable one.
- `TestFresh` is one table: matching files, a changed own file, an added file, a changed `OtherFiles` entry and a changed linked package's file.
- `TestReaderIndex` is one table over the index states: no index, a current index (written by `WriteIndex`), an index file that does not parse,
  an index whose stamp differs after a map was rewritten, and an index written before a map file was added.
  The rewritten map changes a reach pair the index holds, and the row moves the file's modification time with `os.Chtimes`, so a reader that trusted the stale index would answer otherwise.
  Each row's `TestsReaching` answers equal those of the no-index row, and `Open` leaves the index file's bytes as it found them.

`doc.go` gains the reader's rules, the index and its bound.

**ImpactSummary:** Adds the reader beside card 1's finder inside the new package; only its doc.go changes, and nothing outside the package calls it yet.
