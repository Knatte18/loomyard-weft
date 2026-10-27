# A read creates no directory, so hub readers leave half-built pairs alone

`lym fabric add` could stop at Worktree-Create with "warp repo already contains a real _lym … it predates weft" on a brand-new pair.
The cause was a hub-wide reader: the `lym batten overview` window and the tender list the hub's orchs through `orchcli.HubOrchs`.
`HubOrchs` walks every git worktree, including a pair `Topology.Add` is still creating, and reads each origin record.
`state.ReadJSON` ran `os.MkdirAll` on the record's directory, so a poll in the gap between weft creation and junction wiring left a real `<warp>/_lym/fabric`, which the wiring then refused.
`ReadOrigin` also seeded the weft `.weft` lock directory and its exclude, which left a stray `.weft` under a `.fabric-wt-staging-*` directory when a poll caught a pair still in staging.

## What changes

- `state.ReadJSON` creates no directory.
  A record whose directory is absent reads as missing.
  When the lock file's directory is absent, the read goes ahead without the lock.
  `ReadJSONNoMkdir` folds into `ReadJSON`, and every caller moves over.
  A read failure other than not-exist wraps `ErrRead`, and a malformed file wraps `ErrDecode`.
- Readers that created a lock directory only so the read could lock stop doing so: `fabricengine.ReadOrigin`, `websterengine.LoadState`, `loomengine.CheckSeed`, batten's `checkRunStatus` and the shed `status` verb.
  `pause` and `goto` write, so they keep `EnsureStatusLockDir`.
- `HubOrchs` skips a worktree under a fabric staging directory before resolving it.
  It also skips a pair whose `_lym` is not yet a link, before reading its origin.
  Both checks live in fabricengine as `InWorktreeStaging` and `LymJunctionWiredHere`.
- `Topology.Add` wires the junctions right after the weft worktree is created or adopted and moved to its anchor.
  The wiring now runs ahead of the local-branch marks and `writeLaunchers`, so the window in which a warp worktree has no junction is shorter.

## Design decisions

- **Absent lock directory means an unlocked read.**
  No writer can hold a lock in a directory that does not exist, because every writer creates the directory before it locks.
  Every write replaces its file by atomic rename, so an unlocked read still sees a whole file.
  The lock is skipped only on a not-exist error from opening the lock file; any other lock error still fails the read.
  This also lets a tracked `_lym` record be read on a machine whose never-tracked `.lym` lock directory does not exist yet, without a read-side mkdir.
- **The held-`Add` test lives in fabricengine.**
  Holding `Add` from orchcli would need an exported test hook in production code.
  The test holds `Add` through an unexported `beforeWire` seam on `Topology`, set via `export_test.go`.
  While `Add` is held, it runs the reader path `HubOrchs` takes over every listed worktree: `List`, `lymcwd.ResolveWorktree` and `ReadOrigin`.
  A separate orchcli test covers `HubOrchs` itself.
- Only `WriteOrigin` keeps the lock path that creates `.weft` and seeds its exclude.
  `ReadOrigin` joins its lock path onto `.weft` directly.

## Tests

- `TestReadJSON` (`internal/state`) has rows for each case:
  - a missing directory reads as missing and creates nothing;
  - an absent lock directory reads the record and creates nothing;
  - an unreadable path matches `ErrRead`;
  - malformed JSON matches `ErrDecode`.
  `TestMissingFile` and `TestReadJSON_MissingFileCreatesNoDirectory` pinned the old behaviour and are removed.
- `TestAdd_HubReaderBeforeWiringLeavesNoLym` (`internal/fabricengine`, tier1) holds `Add` before wiring and reads every listed worktree's origin.
  It asserts the half-built pair was read, that no `<warp>/_lym` exists at that point, and that `Add` completes with the junction a link.
- `TestHubOrchs_ListsWiredBeamsAndSkipsUnwiredPairs` (`internal/orchcli`, tier1) lists the prime and a wired beam, skips a beam whose `_lym` link was removed, and asserts no `_lym` reappears in it.
- `TestStatusCmd_AbsentLockDirReadsWithoutCreatingIt` replaces the two `EnsureStatusLockDir` arms of the status verb's tests.

## Notes for the reviewer

- `TestEnforcement_MarkdownLinks/repo` (`internal/lymcwd`) fails on this branch's fork point, independent of this change.
  The broken links are in the captured `docs/research/*-bench-2026-10-27/data/prompts/step-1-explore-fan.md` files from main's bench commits.
  Per the parent's decision "Leave the bench link breaks to main", this change touches neither those files nor the link allowlist.
  The task `docs-link-skip-bench-data` fixes the guard on main, and this branch takes that fix in from main.
