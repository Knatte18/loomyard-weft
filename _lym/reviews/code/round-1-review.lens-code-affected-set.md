```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: the set found per change**

- **`BoardWriteLockPath` now returns `(string, error)`.**
  - Set: callers `shortnamecommit.go:19` and `stencilcommit.go:65`.
  - Both callers are updated. Covered.
- **`gitrepo.PushLockFileName` changes to `lym-push.lock` and the lock moves into the git dir, now reached through `Repo.PushLockPath`.**
  - Set: callers in `bolt.go`, `pushclassified.go`, `originreconcilemoves.go`, `push.go`, and the integration tests. These are updated, and `gitrepo/doc.go` and `fabricengine/doc.go` are updated.
  - Not covered: `internal/fabricengine/spawn.go:226` still says the rebase-free push "leaves no untracked .gitrepo-push.lock residue … which has no exclude entry for that file". That name and exclude premise no longer exist.
- **Bolt and board locks move to `<checkout>/.lym/locks/` through `BoltLockPath` and `swapLockPath`.**
  - Set: `board.go`, `sync.go`, `store.go`, `ownerset.go`, `bolt.go`, `heddlecli/verbs.go`, `heddleengine/render.go`.
  - A grep of non-test code found no remaining `board.lock` or `swaplock` path built beside a Bolt file. Covered.
  - Docs `boardengine/doc.go` and `fabricengine/doc.go` follow. `fabricengine/doc.go:748` still names `board.push.lock`, which is still the lock's name, so it is correct.
- **Review-record locks in `shedadapters` move through `TrackedFileLockPath`.** The grep found no other record writer under `_lym` still using `path+".lock"`. Covered.
  - Other sites (`failedpush.go:103`, `corrindex.go`, `mergestate.go`, `fronttask.go:106`, `moverecord.go`) keep `path+".lock"`. I did not confirm that every one of their files sits outside `_lym`.
  - The runtime guard would refuse a lock path under `_lym`, but it does not cover Bolt, and no test scans for a beside-lock built next to a Bolt file.
- **The `*.lock` and `*.swaplock` excludes are removed, the legacy names stay excluded, and the board `.gitignore` is rewritten.**
  - Set: `recordsgit.go`, `boardengine/sync.go`, `weftgit_exclude_test.go`, `boardtest/sync_test.go`, `pattern/PATTERN-fabric-git.md:25`. All are updated. Covered.
- **`PATTERN.md` and `pattern/`.**
  - The new `PATTERN-tracked-file-locks` entry and background file are added, and `PATTERN-durable-vs-ephemeral-state` and `PATTERN-fabric-git` are updated.
  - The new entry's `(test)` claim is only partly backed. See finding 2.
  - The rubric's declared-changes list is empty in the directive, so nothing further is owed there.

**Findings**

1. **LOW, class `consistency`:** `spawn.go:226` still describes a `.gitrepo-push.lock` residue and a missing exclude entry. The push lock now sits in the git dir and the file name is gone. The `PushWarpRebaseFreeAt` doc comment needs its premise dropped or rewritten. Evidence: `internal/fabricengine/spawn.go:224-227`.
2. **MEDIUM, class `design`:** the new `PATTERN-tracked-file-locks` entry says a test refuses a lock path built beside a file under `_lym` or Bolt. The only guard is `lock.refuseTrackedTree` (`internal/lock/tracked.go:869-880`). It matches `_lym` path segments only and returns early on any earlier `.lym` segment. It never matches Bolt, so a lock built beside a Bolt file passes. The test `TestAcquire_LockInTrackedLymDir` checks only a `_lym` path. The Goal's fourth bullet ("refuses a lock path built beside a file under `_lym/` or Bolt") is therefore only half met, and the pattern text overstates the test. Evidence: `internal/lock/tracked.go:869`, `internal/lock/tracked_test.go:650`, `pattern/PATTERN-tracked-file-locks.md:8`, `PATTERN.md` entry.
3. **MEDIUM, class `scope`:** commit `4406aef5` ("PATTERN.md overview back under its headroom line") shortens two unrelated entries. In the `PATTERN-cwd-resolution` overview line it drops "hub containers and workspace files stay hub-reachable only", and `pattern/PATTERN-cwd-resolution.md` has no "hub-reachable" or "hub container" text, so the rule is lost from the constraints. In `PATTERN-durable-vs-ephemeral-state` it drops "instruction files and their `.heddle` temporary directory"; the background file still carries the `.heddle` text, so that trim is harmless. The Goal does not ask to change the `cwd-resolution` entry, and removing a rule from an entry the task did not touch is unlicensed. Evidence: `git diff 040570b9..HEAD -- PATTERN.md`, lines 9-10.

**Tool calls**

1. A bash grep and `sed` batch: stale lock names and docs across `.go`, `.md` and `.yaml`; `pattern/PATTERN-cwd-resolution.md` and the durable-state background; the `failedpush`, `corrindex` and `mergestate` path definitions; `spawn.go:220-230`; the origin of `PATTERN.md`'s trimmed text. My context did not hold any of these.
