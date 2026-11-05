---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/lock/tracked.go:869
    summary: The Goal's check refusing a lock path built beside a file under _lym or Bolt covers _lym only, so a lock beside a Bolt file passes and PATTERN-tracked-file-locks overstates its test
    fix: In internal/lock/tracked.go extend refuseTrackedTree so it also refuses a lock path whose directory is the Bolt checkout root or a tracked directory below it, that is a path under the Bolt directory name with no .lym segment after it; keep the Bolt directory name as one constant in internal/lymdirs if internal/lock may not import lymcwd (check PATTERN-leaf-packages). Add rows to TestAcquire_LockInTrackedLymDir in internal/lock/tracked_test.go for a board.lock and a board.json.swaplock built beside a file in a directory named like the Bolt checkout, refused by every acquirer, and a row for the accepted BoltLockPath form. Keep the wording of pattern/PATTERN-tracked-file-locks.md and its PATTERN.md line true to what the test then refuses.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: PATTERN.md:11
    summary: The headroom commit dropped the clause hub containers and workspace files stay hub-reachable only from the PATTERN-cwd-resolution overview line, an entry the Goal does not touch, and the background file does not carry it
    fix: In PATTERN.md restore the PATTERN-cwd-resolution line to its text before commit 4406aef5, ending "...never import lymcwd; hub containers and workspace files stay hub-reachable only. (test)", and restore "bar heddle's rendered instruction files and their `.heddle` temporary directory at a warp worktree's anchor" on the PATTERN-durable-vs-ephemeral-state line. Regain the headroom by shortening the new PATTERN-tracked-file-locks line or the new bullet in pattern/PATTERN-durable-vs-ephemeral-state.md, never an unrelated entry.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/settlerecord.go:124
    summary: The review-record lock move in shedadapters has no test, so reverting to path plus .lock passes in any test whose temp path has no _lym segment
    fix: Add a table row to the existing settle-record tests in internal/shedadapters, or one test beside them, that writes a settle record, a settle patch, a generation record and a CarryRecords copy under a run directory inside a temp dir holding a _lym directory, then asserts no file ending in .lock or .swaplock sits in the run directory and that the lock files sit under the temp dir's .lym/locks/_lym mirror path.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/lock/tracked.go:872
    summary: refuseTrackedTree returns nil at the first .lym segment anywhere in the path, so a tracked _lym lock under a checkout whose ancestor is named .lym is accepted, and no test pins that choice
    fix: In refuseTrackedTree exempt a path only when a .lym segment is immediately followed by the locks segment, and otherwise keep scanning for _lym; run the verify gate to confirm no ephemeral site under .lym/<other>/ carries a _lym segment. Add a row to TestAcquire_LockInTrackedLymDir for a checkout under a directory named .lym whose _lym lock is refused, and one for the accepted .lym/locks/_lym mirror path.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/spawn.go:226
    summary: The doc comment on the rebase-free push still describes a .gitrepo-push.lock residue and a missing exclude entry, but the push lock now sits in the git dir under another name
    fix: In internal/fabricengine/spawn.go rewrite the sentence at lines 224-227 so it states only that the rebase-free push takes no push lock, and drop the .gitrepo-push.lock residue and exclude-entry premise.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/lock/tracked_test.go:639
    summary: TrackedLockPath symlink resolution, TrackedFileLockPath on a relative path, gitrepo PushLockPath and the lock directory creation in internal/state have no direct row
    fix: Add a TestTrackedLockPath row whose root is a symlink to a directory and wants the resolved root; add a TestTrackedFileLockPath row for a relative path under _lym; add a gitrepo test asserting PushLockPath is lym-push.lock in the git dir and that PushCoalesced creates it there; add a state test row whose lockPath directory is absent for WriteJSON and UpdateJSON.
---

### [BLOCKING] The check against a lock beside a Bolt file is missing

**Location:** `internal/lock/tracked.go:869`, `internal/lock/tracked_test.go:650`, `pattern/PATTERN-tracked-file-locks.md:8`.
**Issue:** The Goal's fourth Idea bullet asks for a check, test or lint, that refuses a lock path built beside a file under `_lym/` or Bolt.
`refuseTrackedTree` matches only the `_lym` segment, and `TestAcquire_LockInTrackedLymDir` only drives `_lym` paths.
A `board.lock` or `.swaplock` built beside a Bolt file still passes every acquirer, and no scan test refuses the old `path + ".lock"` construction.
The new PATTERN entry carries `(test)` and its background says "A test refuses a lock path built beside a file under `_lym` or Bolt", which is true for `_lym` alone.

### [MEDIUM] Unrelated PATTERN rule text dropped by the headroom commit

**Location:** `PATTERN.md:11-13`, from commit `4406aef5`.
**Issue:** The commit that restored the overview's headroom cut the `PATTERN-cwd-resolution` clause "hub containers and workspace files stay hub-reachable only", which `pattern/PATTERN-cwd-resolution.md` does not repeat (it states only the workspace-file half at line 46).
It also narrowed the `PATTERN-durable-vs-ephemeral-state` exception from heddle's rendered instruction files and `.heddle` directory to "rendered files".
The Goal licenses neither change, and a rule weakened in an entry the task did not touch is a scope defect.

### [MEDIUM] Review-record lock move has no test

**Location:** `internal/shedadapters/settlerecord.go:124`, `:133`, `:152`, `:258`.
**Issue:** The diff moves four record-lock sites to `lock.TrackedFileLockPath` and adds no test under `internal/shedadapters`.
Only the runtime refusal in a test that happens to run under a `_lym` path would catch a revert, so the behaviour has no row that fails when it breaks.

### [LOW] The tracked-tree refusal is skipped after any earlier .lym segment

**Location:** `internal/lock/tracked.go:872`.
**Issue:** The loop returns nil at the first `.lym` segment of the whole path, before a following `_lym` is examined.
A checkout under a directory named `.lym` therefore keeps its `_lym` lock beside the file unrefused.
No test pins the choice either way.

### [LOW] Stale comment about a push lock residue

**Location:** `internal/fabricengine/spawn.go:226`.
**Issue:** The comment still names `.gitrepo-push.lock` and an absent exclude entry; the lock is now `lym-push.lock` in the git dir.

### [LOW] Rows missing for four moved behaviours

**Location:** `internal/lock/tracked_test.go:599-648`, `internal/gitrepo/push.go:31`, `internal/state/state.go:61`.
**Issue:** `resolvedTempDir` hands the lock tests an already-resolved root, so removing the symlink resolution fails no row.
No row runs a relative path, `PushLockPath`, or a `lockPath` whose directory is absent.

## Rejected

- Upgrade overlap, where an old and a new binary hold different lock files for one board (lens:code-correctness): the Goal asks for the move and no drain step, and mixed binaries on one board are not a supported state.
- Board `.gitignore` idempotence and no-trailing-newline rows (lens:code-tests): `stepSyncGitignoreIgnoresManifestAndDropsLockPatterns` pins the exact resulting content, and the code-correctness run showed both inputs correct.
- Board sync lock-location assertion sitting in the integration tier (lens:code-tests): the tier follows the fixture, which needs a real git remote.
- Bolt `.lym/locks` being swept into a stage-all board commit: Bolt is the weft main worktree, and `seedWeftArtifactExcludes` excludes `.lym/` in the common git dir.
