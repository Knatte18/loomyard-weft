# Round 1 fixer-report

## Changes

- **F1** fixed as proposed, with one addition.
  `refuseTrackedTree` in `internal/lock/tracked.go` now also refuses a lock path inside a `_bolt` directory with no `.lym` segment after it.
  The Bolt name is `lymdirs.BoltDirName`; `lymcwd.BoltDirName` re-exports it.
  `internal/lock` cannot import `lymcwd`, which would close the import cycle `dotgit` test, `tmuxkit`, `lymtemp`, `lock`, `lymcwd`, `dotgit`, so the declaration moved.
  The `_bolt` owner row of `TestEnforcement_GeometryLiterals`, the `lymcwd` leaf allowlist and `PATTERN-cwd-resolution.md` follow it.
  Addition: a `_bolt` counts as a Bolt checkout only when it holds a `.git` entry.
  Without it the board's push lock, `<bare>/worktrees/_bolt/lym-push.lock`, was refused and tier1 failed in hub setup.
  `TestAcquire_LockInTrackedLymDir` became `TestAcquire_LockInTrackedTree`, with refused rows for `board.lock`, `board.json.swaplock` and a nested Bolt directory, and accepted rows for `TrackedLockPath`, a `.lym/pause` lock under Bolt and the git-dir entry.
  Docs updated: `lock.go` and `tracked.go` headers, `lymdirs` headers, `pattern/PATTERN-tracked-file-locks.md`.
- **F2** fixed differently.
  Proposed: restore both clauses to their text before `4406aef5` and regain the headroom from the new lock entry or the new durable-state bullet.
  Not done as written: the overview is capped at 15564 bytes (base 15464), so the new entry may add 100 bytes, and an entry with its mandatory background link alone is about 125.
  The durable-state bullet is in `pattern/`, not the overview, so cutting it regains nothing.
  Done: the cwd clause "hub containers and workspace files stay hub-reachable only" is restored in full; the heddle clause reads "heddle's instruction files and `.heddle`" (dropped "rendered", "their", "temporary directory"); the lock entry reads "Tracked-file lock: path via `lock`. (test)" with its link.
  Dropping the link fails `orphan-background` (`TestOverview_RealFilePassesCheck`).
  The overview is 15563 bytes.
- **F3** fixed as proposed.
  `TestRecordLocks_SitOutsideTheTrackedRunDirectory` in `internal/shedadapters/settlerecord_test.go` writes a settle record, patch and generation record, reads a record, and carries them; it asserts each lock sits under `.lym/locks/_lym/...` and no `.lock` or `.swaplock` file sits in either run directory.
- **F4** fixed as proposed.
  Folded into the `refuseTrackedTree` rewrite: `.lym` exempts only when followed by `locks`.
  Row `.lym/checkout/_lym/record.json.lock` refused; the mirror path accepted by the `TrackedFileLockPath` row.
  No ephemeral site under `.lym/<other>/` carries a `_lym` segment: the tier1 run over the affected packages is green.
- **F5** fixed as proposed in `internal/fabricengine/spawn.go`.
  Self-check follow-up: `internal/gitrepo/push_test.go` still locked `.gitrepo-push.lock`, so its crash step no longer held the lock `PushCoalesced` takes; both sites now use `PushLockPath`.
  Stale comments about the old file in `push_test.go`, `internal/fabriccli/cli_test.go` and `coalesce_integration_test.go` were corrected.
- **F6** fixed as proposed.
  New rows: symlinked and absent roots in `TestTrackedLockPath`, a relative path in `TestTrackedFileLockPath`, `internal/gitrepo/pushlock_test.go` for `PushLockPath` and the lock `PushCoalesced` creates there, `internal/state/lockdir_test.go` for an absent lock directory under `WriteJSON`, `WriteFile` and `UpdateJSON`.

Gate repair: the verify gate failed `TestEnforcement_BoltDirNameDeclaredOnce` in `internal/lymcwd`, which still expected the Bolt name's one spelling in `anchor.go`.
The declaration moved to `internal/lymdirs/dirs.go`, so the test's `declarationFile` and its comment now name that file.
`lym gate test` over `lymcwd`, `lymdirs`, `lock` and `pattern` ended green, as did `--tier 1` over the touched and dependent packages earlier.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: `lock.go` package doc still described a `_lym`-only refusal. Fixed as proposed.
- fix-consistency, LOW: `lymdirs` headers said "the two" tokens. Fixed as proposed.
- fix-consistency, LOW: overview heddle clause differs from its background. Fixed differently, see F2; the headroom forces the shorter wording.
- fix-consistency, LOW: overview lock entry and background disagree on scope and the `.git` condition. Background updated with the condition; the overview stays short by the headroom.
- fix-consistency, LOW: `lymcwd` import allowlist and the `lymdirs` doc. Same as the `lymdirs` header finding; fixed.
- fix-proposal, MEDIUM: `TrackedFileLockPath` yields a refused path for a Bolt file. Fixed differently: the background now names `TrackedLockPath` for a Bolt file and `TrackedFileLockPath` for a `_lym` file; `TrackedFileLockPath` does not gain a Bolt mirror, which the review did not ask for.
- fix-proposal, LOW: the Bolt refusal depends on a `.git` entry. Fixed; the background states it.
- fix-proposal, LOW: a `_lym` lock under a checkout nested below `.lym/locks` is accepted. Disputed, see below.
- fix-proposal, LOW: F2 text differs from the review. Same as the F2 entry; fixed differently.
- fix-affected-set, LOW: stale `.gitrepo-push.lock` test comments. Fixed, and the stale lock path in `push_test.go` found beside them.
- fix-affected-set: four path origins not traced (`mergeStatePath`, `fronttask.lockPathOf`, `innerRunMemoryLockFile`, `failedpush.go:103`); the green tier1 run over `fabricengine` covers the first and last, the others are covered by the run-time refusal.

## Disputed

- fix-proposal LOW, nested checkout below `.lym/locks`: ground `every fix worse`.
  `TrackedFileLockPath` mirrors a file by its path below the nearest `_lym` above it, so a mirror path below `.lym/locks` can itself hold a `_lym` segment (`rel` joins deeper directories, `internal/lock/tracked.go` loop).
  Refusing `_lym` after `.lym/locks` would refuse such a legitimate lock.

## Deferred

Nothing deferred.
