# Card 5 — teardown-lock-add-discard-push

**Edit:**
- `internal/fabricengine#Topology.lockPairTeardown` -> `func (t *Topology) lockPairTeardown(rec *Mutations, l *lymcwd.Location, slug string, try bool, rerun string) (release func(), err error)`
- `internal/fabricengine#TeardownLockPath`
- `internal/fabricengine#ErrPairPushRunning`
- `internal/fabricengine#Topology.removePair`
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#Topology.DiscardTaskBranch`
- `internal/fabricengine#PushPairAnchored`
- `internal/loomcli#commitRecordsVerb`
- `internal/loomcli#TestCommitRecordsVerb_Success`
- `internal/fabriccli#addWeftVerbs`

**Create:**
- `internal/fabricengine_test#TestTeardownLock_SerialisesAddDiscardAndPairPush`
- `internal/fabriccli_test#TestRunCLI_PushSkippedWhileTeardownLockHeld`

**Extends:**
- `internal/fabricengine/doc.go`
- `internal/fabriccli/doc.go`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/fabricengine#SetTeardownWaitForTest`
- `internal/fabricengine_test#requireNothingCreated`
- `internal/fabricengine_test#discardHub`

**Intent:** Decision record D7, its first, second, fourth and fifth bullets: a slug's pair is made and its task branch discarded under the slug's teardown lock exclusive, and every synchronous push of the pair's weft branch takes it shared.

- `lockPairTeardown` gains `rerun`, the re-run its wait-out refusal names; `Topology.removePair` passes today's text (`re-run \`lym fabric remove <slug>\``), so `Remove`'s refusal is unchanged.
  The refusal still wraps `ErrPairPushRunning` and names the slug's running push or removal.
- `Topology.add`, for `Add` and not `AddLocal`, takes the lock exclusive with `try` false right after slug validation and before its origin probes, holds it to its return, rollback included, and waits up to `t.teardownWait`, as `Remove` waits.
  A wait that runs out refuses before any mutation with an error wrapping `ErrPairPushRunning` naming the slug's running push or removal and `re-run the command that made the pair once it finishes`.
  Its callers spawn the new pair's push after it returns, as today, so the push child takes the lock shared once `Add` has released it.
- `Topology.DiscardTaskBranch` takes the lock exclusive the same way before its first tip read and holds it across its archive and remote delete; its wait-out refusal names `re-run the command that discarded the task branch once it finishes`, since fabric names none of its callers' verbs (`PATTERN-fabric-isolation`).
- `PushPairAnchored`, the synchronous push `commit-records`, `lym fabric push` and the landing's `PushTask` (`internal/loomcli/landingdeps.go`) run, takes the pair's teardown lock shared through `lock.TryAcquireReadLock(TeardownLockPath(hub, slug))` for a non-prime pair (the same slug `statusPushRequest` names) before the weft push lock and holds it through the push.
  When any holder has it exclusive (a removal, an `Add`, a discard or a cleanup sweep of the slug), it pushes nothing and returns an empty `PushResult` and an error wrapping `ErrPairPushRunning` naming the slug and that the commits stay local for the pair's next push.
  The prime's records push takes no teardown lock, as today.
- `commitRecordsVerb` reads a push error matching `ErrPairPushRunning` as a skip: it logs a Warn and returns the ok envelope `committed` true with `push_skipped` carrying the error's text.
  `internal/loomcli/doc.go` gains a sentence on `lym loom commit-records`: a push skipped while the slug's teardown lock is held exclusive is an ok envelope with `push_skipped`, its commit left local for the pair's next push.
  `addWeftVerbs`' `lym fabric push` does the same, returning its ok envelope with `push_skipped`; `internal/fabriccli/doc.go`'s `lym fabric push` envelope line gains `push_skipped` beside `local_skipped`.
  The landing's `PushTask` returns the error unchanged, so a landing never reads a skipped push as the task pair pushed and fails its push step instead.
- Bound: `Add` and the discard wait at most the teardown wait and never on another slug's work; fetches and origin judges stay outside the lock.
  A hand `lym fabric add`, a cancel's discard or batten's adopt-only `Add` meeting a removal's hold is refused before any mutation; the commits a skipped synchronous push leaves stay local, as a detached push's do, and are given up with a removal.
  A skipped push is never read as a landed one: only `commit-records` and `lym fabric push` treat the sentinel as a skip, and the landing's push fails on it.
- Update the doc comments that name the lock's holders: `teardownlock.go`'s file header, `TeardownLockPath`'s ("excludes a removal … against the pair's detached pushes"), `ErrPairPushRunning`'s ("the sentinel Remove's refusal wraps"; it also names the skipped synchronous push), `PushPairAnchored`'s (the shared hold and the skip), `commitRecordsVerb`'s (the skip envelope), `Topology.Add`'s (the hold and its wait-out refusal), and extend `internal/fabricengine/doc.go`'s teardown-lock paragraph with these holders.

Test `TestTeardownLock_SerialisesAddDiscardAndPairPush`, `tier1`, parallel: `SetTeardownWaitForTest` sets the wait on the one Topology, and each row has its own hub, so no process-global state is touched; the test holds the lock itself with `lock.AcquireWriteLock(TeardownLockPath(...))` and releases it from a goroutine, every ordering told by channels:
- `Add` of the slug blocks while the lock is held, then makes the pair once it is released.
- `Add` whose shortened wait runs out returns an error wrapping `ErrPairPushRunning` whose text names the re-run, and `requireNothingCreated` holds.
- `DiscardTaskBranch` (`discardHub`) blocks while the lock is held and archives and deletes once it is released.
- `DiscardTaskBranch` whose shortened wait runs out returns an error wrapping `ErrPairPushRunning`, and the task branch stands locally and on origin with no archive tag pushed.
- `PushPairAnchored` of a pair with a new records commit, run while the lock is held exclusive, returns an error wrapping `ErrPairPushRunning`, and origin's weft tip is unchanged.
- `PushPairAnchored` at the prime while a pair's teardown lock is held exclusive pushes the prime's new records commit.

`TestCommitRecordsVerb_Success` gains a row whose fake `Push` returns an error wrapping `fabricengine.ErrPairPushRunning`: the envelope is ok with `committed` true and `push_skipped` set (D7's `commit-records` criterion).

Test `TestRunCLI_PushSkippedWhileTeardownLockHeld`, `tier1`, parallel, over a `hubforge` hub with an ordinary pair added through `lym fabric add` and a new records commit in its records worktree: the test holds the slug's teardown lock exclusive with `lock.AcquireWriteLock(TeardownLockPath(...))`, runs `lym fabric push` from the pair's code worktree, and asserts an ok envelope with `push_skipped` set and origin's weft tip unchanged.

**ImpactSummary:** Re-signs one private lock helper (single caller `removePair`) and adds a lock to `Add`, the discard and the synchronous pair push; callers in battencli, fabriccli and loomcli keep their signatures and gain only a bounded wait or a skipped push while an exclusive holder runs, which `commit-records` and `lym fabric push` report as ok and the landing reports as a failed push.

**Commit:** `5: Add, the discard and synchronous pair pushes take the slug's teardown lock`
