# Card 7 — The hub config walk after a binary change

**Create:**
- `plan:internal/hubreconcile#Geometry` -> `type Geometry struct`
- `plan:internal/hubreconcile#Geometry.StampPath` -> `func (g Geometry) StampPath() string`
- `plan:internal/hubreconcile#Geometry.LockPath` -> `func (g Geometry) LockPath() string`
- `plan:internal/hubreconcile#Options` -> `type Options struct`
- `plan:internal/hubreconcile#DefaultLockWait` -> `const DefaultLockWait = 2 * time.Minute`
- `plan:internal/hubreconcile#Ensure` -> `func Ensure(geom Geometry, opts Options) error`
- `plan:internal/hubreconcile#LockTimeoutError` -> `type LockTimeoutError struct`
- `plan:internal/hubreconcile#LockTimeoutError.Error` -> `func (e *LockTimeoutError) Error() string`
- `plan:internal/hubreconcile#WorktreeError` -> `type WorktreeError struct`
- `plan:internal/hubreconcile#WorktreeError.Error` -> `func (e *WorktreeError) Error() string`
- `plan:internal/hubreconcile#WorktreeError.Unwrap` -> `func (e *WorktreeError) Unwrap() error`
- `plan:internal/hubreconcile#BuildKey` -> `func BuildKey(id buildvcs.Identity, fingerprint string) string`
- `plan:internal/hubreconcile#stamp` -> `type stamp struct`
- `plan:internal/hubreconcile#readStamp` -> `func readStamp(path string) (stamp, bool, error)`
- `plan:internal/hubreconcile#writeStamp` -> `func writeStamp(path string, s stamp) error`
- `plan:internal/hubreconcile#walkHooks` -> `type walkHooks struct`
- `plan:internal/hubreconcile#ensure` -> `func ensure(geom Geometry, opts Options, hooks walkHooks) error`
- `plan:internal/hubreconcile#reconcileHubWide` -> `func reconcileHubWide(boardDir, primeAnchor, revision string) error`
- `plan:internal/hubreconcile#reconcileWorktree` -> `func reconcileWorktree(boardDir string, w fabricengine.CodeWorktree, revision string, hooks walkHooks) (skipped bool, err error)`
- `plan:internal/hubreconcile#restoreConfig` -> `func restoreConfig(prior map[string][]byte) error`
- `internal/hubreconcile/doc.go`
- `internal/hubreconcile/export_test.go`
- `internal/hubreconcile/stamp_test.go`
- `internal/hubreconcile/ensure_integration_test.go`
- `internal/hubreconcile/seam_enforcement_test.go`
- `internal/hubreconcile/testmain_test.go`
- `internal/hubreconcile/testmain_integration_test.go`

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-told-geometry.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/buildvcs#Running`
- `plan:internal/configreg#Fingerprint`
- `plan:internal/configsync#FileError`
- `plan:internal/configengine#ScratchDir`
- `plan:internal/fabricengine#CodeWorktrees`
- `plan:internal/fabricengine#OpenCodeWorktree`
- `plan:internal/fabricengine#Fabric.MergeBlocked`
- `plan:internal/fabricengine#PairCompleteAt`
- `internal/configsync#ReconcileAll`
- `internal/configsync#ReconcileHubWideAt`
- `internal/fabricengine#Fabric.Commit`
- `internal/fabricengine#Bolt.CommitWritten`
- `internal/fabricengine#Bolt.Push`
- `internal/fabriccli#runReconcile`

**Intent:**
Decisions 1, 2, 5, 6, 7, 8 and 9: one new non-CLI package, `internal/hubreconcile`, whose one entry point `Ensure` reconciles a hub's config once after a binary change, under a hub lock, and commits everything it wrote.
It is told the hub's geometry and derives none (`PATTERN-told-geometry`): it imports `internal/buildvcs`, `configengine`, `configreg`, `configsync`, `fabricengine`, `fsx`, `lock` and `logger`, never `lyxcwd`.
Members go in `hubreconcile.go` (geometry, options, errors, `Ensure`), `stamp.go` (build key and stamp) and `walk.go` (the walk).

Geometry and stamp (Decision 2):
- `Geometry` carries `BoardDir` (the hub's board dir, where the hub-wide config lives at `_lyx/config/`) and `WorktreePath` (the caller's code worktree, from which the walk lists the hub's code worktrees).
- `StampPath` is `configengine.ScratchDir(BoardDir)/build-stamp.json` and `LockPath` is `configengine.ScratchDir(BoardDir)/reconcile.lock`, so stamp and lock sit side by side under `<BoardDir>/.lyx/config/`.
- `BuildKey` returns the hex SHA-256 of the revision, the modified flag and the registry fingerprint, each length-prefixed (Decision 1).
- `stamp` is the file's JSON: `build_key`, plus `vcs_revision` and `vcs_modified` in readable form for the log.
- `readStamp` reports an absent file as not found with no error; `writeStamp` creates the dir and writes atomically through `fsx`.
- The running key is `BuildKey(buildvcs.Running(), configreg.Fingerprint())`, and an absent stamp counts as stale.

`Ensure(geom, opts)` calls `ensure` with zero hooks.
`Options` carries `LockWait` (the hub-lock bound; zero means `DefaultLockWait`) and `Pair` (a code worktree root; see the pair call below).

`ensure`, the hub walk (Decision 5):
1. Read the stamp; a stamp naming the running key returns nil at once: one file read, no lock, no git.
2. Take the hub lock with `lock.AcquireWriteLockWithin(LockPath, wait)`, creating its dir first; a lock not acquired within the wait returns `*LockTimeoutError` and touches nothing.
3. Re-read the stamp under the lock; a fresh one releases and returns nil.
4. List the code worktrees with `fabricengine.CodeWorktrees(geom.WorktreePath)` to find the prime (`Main`).
5. `reconcileHubWide`, then `reconcileWorktree` on the prime.
6. List again and `reconcileWorktree` each pair not yet walked; call `hooks.afterEnumerate` after each listing.
   Repeat until a listing finds no new pair, so a pair registered while the walk ran is walked before the stamp.
7. Write the stamp only when no worktree was skipped as mid-merge, then release the lock.
   A skipped worktree leaves the stamp stale, logged, so the next start verb retries the walk.

`DefaultLockWait` is two minutes: well above a walk with no contended pair lock.
The holder's own per-worktree commits wait on each pair's combined write lock as `Fabric.Commit` does today, unbounded (Decision 5).

`reconcileHubWide` follows `fabriccli`'s `runReconcile` precedent:
- It runs `configsync.ReconcileHubWideAt(boardDir, primeAnchor, true)` inside `fabricengine.NewBolt(boardDir).CommitWritten`, returning the `ConfigFileRel` of every applied module and pruned legacy file.
- It then calls `Bolt.Push`; a push failure is logged with `logger.Warn` and never fatal (Decision 7).
- A reconcile failure, or a commit failure, returns a `*WorktreeError` for the board dir.

`reconcileWorktree` handles one code worktree:
1. For a pair (not `Main`), ask `PairCompleteAt(w.Path)`; a pair that is not complete is skipped and logged with the reason, before anything is opened or written, and is not reported `skipped`.
   This keeps the walk out of a pair `Add` is still creating, where writing a template would leave a real `_lyx` dir where `Add` wires its junction and make `Add` refuse, and out of one a killed `Add` left partway.
   The skip is stamp-neutral, unlike the mid-merge skip: a pair `lyx batten run` is creating gets card 10's pair call once `Add` completes, and a `lyx fabric add` pair is Decision 4's bounded escape; a crashed partial pair never keeps every later start verb walking.
   An `*ErrMissingPath` skips it as removed; any other error returns a `*WorktreeError` for the worktree with no file.
2. Open it with `OpenCodeWorktree`; an `*ErrMissingPath` skips it as removed (not a failure, and it does not keep the stamp stale).
3. A worktree `MergeBlocked` reports mid-merge is not written at all: it is logged and reported `skipped`.
4. Snapshot the prior bytes, or absence, of every registry module's `ConfigFile(w.Anchor, name)`.
5. Run `configsync.ReconcileAll(w.Anchor, boardDir, true)`.
   A failure returns a `*WorktreeError` naming the worktree and the `*configsync.FileError`'s module and path, unless the worktree root has vanished meanwhile, which skips it as removed.
6. Log one `logger.Info` line per changed module (worktree, module, added, removed, migrated) (Decision 8).
7. Commit the worktree-relative paths of every applied result through `Fabric.Commit(files, "lyx: reconcile config for build <revision>", nil, SyncOptions{})`.
   The list holds only files reconcile wrote or removed, so a retired copy's deletion is staged too, and a driver's in-flight files are never swept.
   `Fabric.Commit` keeps its async push rule (`PATTERN-push-both-sides`).
8. `hooks.beforeCommit` runs between the write and the commit.
   A commit that refuses with `*ErrMergeInProgress` or `*ErrForeignMergeState` calls `restoreConfig` on the snapshot, so the files return to their prior bytes, and reports `skipped`.
   Any other commit failure returns a `*WorktreeError` for the worktree with no file.

`ensure` logs one summary line naming the stamp's previous revision and the running one.
Nothing is written to stdout.

The pair call: when `opts.Pair` is set, `ensure` skips both stamp reads and the stamp write, takes the hub lock the same way, finds the `CodeWorktree` whose `Path` equals `filepath.Clean(opts.Pair)` in a fresh listing (card 5 lists each `Path` in that cleaned OS-separator form), and runs `reconcileWorktree` on it alone, whatever the stamp says (Decision 4).
A pair not in the listing is an error naming the path.
Bound: this call reconciles and commits only that one pair's `_lyx/config` files, under the same lock and with the same mid-merge refusal as the walk.

Errors (Decision 9):
- `WorktreeError` carries `Worktree`, `File`, `Module` and `Err`.
- Its message names the worktree, the file and the cause, and ends with the way forward `fix <file> (or run "lyx config <module>" on it), then re-run this command`, or just `re-run this command` when no file failed.
- `LockTimeoutError` carries `Path` and `Wait`.
- Its message says another lyx command holds the hub's reconcile lock and ends with the way forward `re-run this command once it finishes`.
- Neither message names a denied command (`PATTERN-no-denied-recovery`).
- Worktrees that reconciled and committed before a failure keep their commits.

`doc.go` documents:
- the package's purpose and its one entry point;
- the stamp, lock and key;
- the walk order;
- the incomplete-pair skip, which leaves the stamp alone;
- the mid-merge skip and restore;
- the pair call;
- the error shapes;
- the escape Decision 4 bounds: a `lyx fabric add` whose fork read the prime before a walk committed it and that became enumerable only after the walk's last listing, which Loom-Preflight's retired-key refusal backs up;
- the two Open risks it accepts: a downgrade flip-flop, and a lyx-authored records commit in a running pair.

`PATTERN.md` gains, in the same commit, under "CLI" after `PATTERN-config-strictness`, a `PATTERN-binary-change-reconcile` entry: reconciling a hub's config after a binary change happens only through `hubreconcile.Ensure`, which owns the build stamp and the hub lock.
`pattern/PATTERN-told-geometry.md`'s bound-package list gains `hubreconcile`.
`docs/overview.md`'s module tree gains an `internal/hubreconcile/` line ("the hub config walk a start verb runs once after a binary change: build stamp, hub lock, reconcile and commit of every worktree's config") beside `internal/preflightshed/`.

Tests:
- `seam_enforcement_test.go` pins the import allowlist above through `scankit.AssertImportAllowlistNoStale`, so a direct `lyxcwd` import fails.
- `testmain_test.go` and `testmain_integration_test.go` follow `PATTERN-test-isolation`, like `internal/preflightshed`'s pair.
- `export_test.go` exposes `ensure` with hooks to the external test package (`hubreconcile_test`), which `ensure_integration_test.go` needs because `internal/hubforge` imports `internal/hubgeom`, which imports this package from card 8 on.

`stamp_test.go` (untagged):
- `BuildKey` differs on a changed revision, a flipped modified flag and a changed fingerprint, and is equal for equal inputs.
- `readStamp` on an absent file reports not found with no error.
- A `writeStamp`/`readStamp` round trip keeps all three fields.
- `StampPath` and `LockPath` are exactly `<BoardDir>/.lyx/config/build-stamp.json` and `.../reconcile.lock`.

`ensure_integration_test.go` (integration, `internal/hubforge`; each row writes and commits its fixture config on the worktree's records side first):
- An absent stamp, with a retired `master_base` key in the prime's and a pair's `batcher.yaml`, removes both keys and leaves both records worktrees clean.
  The commits land on both records branches, the stamp carries the running key, and a second `Ensure` lands no commit.
- A stamp with the running key leaves a retired key in place and lands no commit.
- A pair with a fabric merge record, and one with foreign `MERGE_HEAD` state, keep their files byte-identical while the prime is committed.
  The stamp stays absent, and after the merge state is cleared the next `Ensure` reconciles that pair and stamps.
- A `beforeCommit` hook that creates a merge record leaves the hooked worktree's files at their prior bytes, with no uncommitted `_lyx/config` change and no stamp.
- An unparseable pair `loom.yaml` returns a `*WorktreeError` naming that pair and file, with `lyx config loom` in its message, and no stamp.
  After the file is fixed the next `Ensure` stamps.
- A held `LockPath` with a 50 ms `LockWait` returns a `*LockTimeoutError` and leaves the stamp absent.
- Two concurrent `Ensure` calls on a stale hub land exactly one reconcile commit on each records branch.
- An `afterEnumerate` hook that adds a pair after the first pair listing gets that pair reconciled and committed before the stamp is written.
- A pair whose directory is deleted before `Ensure` is skipped and the stamp is written.
- A pair whose warp `_lyx` junction has been removed, as `Add` leaves a pair before wiring it, is listed but left unwritten: no `_lyx` path exists at the junction's place after `Ensure`, so `Add`'s wiring would not refuse, no commit lands on that pair's records branch, and the stamp is written.
- A board remote pointed at a missing path still returns nil with the hub-wide change committed in `_board`.
- `Options{Pair: path}` with a fresh stamp, `path` joined from the hub dir and the pair's slug the way `lyx batten run` builds it, still removes that pair's retired key and commits it.

**ImpactSummary:** A new package with no caller yet, and the `PATTERN.md`, PATTERN background and overview entries that land with it.

**Commit:** 7: hubreconcile: reconcile and commit a hub's config once after a binary change
