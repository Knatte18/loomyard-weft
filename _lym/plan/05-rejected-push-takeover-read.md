# Card 5 — rejected-push-takeover-read

**Create:**
- `plan:internal/fabricengine#MachineNames.Holds` -> `func (n MachineNames) Holds(name string) bool`
- `plan:internal/fabricengine#TakeoverError` -> `type TakeoverError struct`
- `plan:internal/fabricengine#TakeoverError.Error` -> `func (e *TakeoverError) Error() string`
- `plan:internal/fabricengine#ErrHandedOver` -> `var ErrHandedOver = errors.New("fabricengine: the run was handed over to another machine, so nothing more is pushed for it")`
- `plan:internal/fabricengine#ReadUpstreamTakeover` -> `func ReadUpstreamTakeover(l *lymcwd.Location, weftPath, slug string) (entry shedengine.Takeover, found bool, err error)`
- `plan:internal/fabricengine#HandOverMark` -> `type HandOverMark struct`
- `plan:internal/fabricengine#HandOverMarkPath` -> `func HandOverMarkPath(hubPath, slug string) string`
- `plan:internal/fabricengine#ReadHandOverMark` -> `func ReadHandOverMark(hubPath, slug string) (mark HandOverMark, found bool, err error)`
- `plan:internal/fabricengine#takeoverStop` -> `func takeoverStop(l *lymcwd.Location, weftPath, slug string, here MachineNames) error`
- `plan:internal/fabricengine_test#TestRejectedRecordsPush_ReadsTheTakeover` -> `func TestRejectedRecordsPush_ReadsTheTakeover(t *testing.T)`

**Edit:**
- `internal/fabricengine#retryWeftPush` -> `func retryWeftPush(rec *Mutations, hubPath, weftPath, slug string, here MachineNames) (deferred bool, err error)`
- `internal/fabricengine#rejectionRetryKind`
- `internal/fabricengine#pushClassified`
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#coalescePushSides` -> `func coalescePushSides(req PushSpawn, opts SyncOptions, here MachineNames, caller string, acquire func(lockPath string) (*lock.FileLock, bool, error)) (res PushResult, err error)`
- `internal/fabricengine#CoalescePushBothAt`
- `internal/fabricengine#CoalescePushRequest` -> `func CoalescePushRequest(req PushSpawn, opts SyncOptions, here MachineNames) (res PushResult, err error)`
- `internal/fabricengine#DetachedPush` -> `func DetachedPush(hubPath string, req PushSpawn, here MachineNames) (PushResult, error)`
- `internal/fabricengine/doc.go`
- `internal/fabriccli#addWeftVerbs`
- `internal/fabricengine_test#TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks`
- `internal/fabricengine_test#TestPushFailureClasses`
- `internal/fabricengine#TestCoalescePushRequest_CoveredCallDefers`
- `internal/fabricengine_test#TestRemove_TeardownLockAgainstThePairsPush`
- `internal/fabricengine#TestDetachedPush_CoveredChildExitsBeforeItsFlags`

**Uses:**
- `internal/testkit/takeoverkit#WriteTakeover`
- `internal/fabricengine_test#newPairFixtureAt`
- `internal/fabriccli/doc.go`

**Intent:**
A rejected records push reads origin for a takeover before any replay (D1, D2).
`retryWeftPush` takes the run's slug and the caller's names, and right after its fetch of the records branch, before the write lock and the replay, it runs `takeoverStop`.
`takeoverStop` does nothing for a zero `MachineNames` or an empty slug.
Otherwise it reads the run's own record from the fetched upstream tip through `ReadUpstreamTakeover`.
- No entry, or an entry whose machine `Holds` reports as this machine's (current or former name): nil, and the replay and push run as today.
  This is the shared-branch case (the records branch of a prime holding other runs' commits) and the adopter's own record after `lym machine name`.
- An entry naming another machine that this machine's hand-over mark of the slug names (same machine): an error wrapping `ErrHandedOver`.
  The mark matches on the machine alone, since a Finalize hand-over (card 11) writes its mark from the Bolt's owner before any entry reaches origin;
  an adopt that names this machine drops its marks of the slug (D2, adopt's part), so a run that comes back is never skipped by a stale mark.
- Any other entry naming another machine: a `*TakeoverError` carrying the slug, the entry and the records worktree.

`rejectionRetryKind` names both as non-retryable, so `settleRejection` returns them at once;
`pushClassified` returns a `*TakeoverError` or an error wrapping `ErrHandedOver` from `settleRejection` as it is, before `classifyFailure` and `fail`, so no failed-push record or notice is written, and `pushSideDetached` returns it rather than logging it as a recorded failure: a takeover is no failed push.
`TakeoverError.Error` names the slug and the new owner machine; `Unwrap` is not needed, callers match it with `errors.As`.

`ReadUpstreamTakeover` reads, at the records worktree's fetched upstream tip, the status file of `slug` under the anchor that tip records (`treeAnchor` over `RepoWiredNames(l)`), falling back to the legacy `self` run directory, as `originStatusAtTip` does.
It returns found false when the worktree has no upstream or the tip holds no record or no entry.
In a pair the record is the child record; on the prime's records branch it is the batten record of the run slug the caller names.

`HandOverMarkPath` is `<HubScratchDir>/<lymdirs.HandOverDirName>/<slug>.json`; `HandOverMark` holds the takeover entry's `Machine` and `At` and the time it was written.
`ReadHandOverMark` reads it tolerantly; an absent mark reads not found (`PATTERN-lym-files`: a cache an absent copy of reads empty).
Its writer comes with the hand-over (card 6).

The callers hand the names and slug in:
`PushPairAnchored` passes `push.Here` and, for a pair, the pair's slug, for the prime `push.RunSlug`.
`coalescePushSides` takes `here` and passes the pair's slug from the records worktree's name, or the request's `PauseSlug` for the prime;
so a rejected detached records push that meets a takeover pushes no code side and returns the takeover, which the child logs (D1).
`CoalescePushBothAt` passes a zero `MachineNames`: it serves a fixture's push and reads no takeover.
`CoalescePushRequest` and `DetachedPush` take `here`, and `lym fabric push` in `addWeftVerbs` builds it from `machinename.Names` in both branches: the detached child's, and the synchronous one, which also sets `RunSlug` to the pair's slug, so the way forward card 1 names never replays over a takeover.
The fabric doc gains the takeover read, the two errors and the hand-over mark.

`TestRejectedRecordsPush_ReadsTheTakeover` (tier1) builds a pair with `newPairFixtureAt`, plants a records commit on origin from a second clone carrying a takeover entry written with `WriteTakeover`, and adds a local transition commit and a code commit. Rows:
- entry naming another machine: `PushPairAnchored` returns a `*TakeoverError`, origin's records and code tips are unchanged, no replay commit exists locally, no failed-push record stands;
- the anchor change's push: `CoalescePushRequest` over the request `statusPushRequest(pair, "")` builds, the one `AnchorChange.Finish`'s `SpawnStatusPush` hands its child: origin's code branch unchanged (D1's anchor-change criterion).
  The child itself cannot run in a test binary (`SpawnDetachedPush` returns `ErrSpawnUnderTest`, `spawn.go:71`), so the row runs the child's body over the child's own request;
- entry naming this machine's former name: the replay runs and both sides land;
- a mark naming the entry's machine, written by the test as JSON at `HandOverMarkPath` (the writer comes in card 6): `ErrHandedOver`, nothing pushed;
- a mark naming another machine than the entry's: a `*TakeoverError`, the mark not matching;
- a zero `MachineNames`: the replay runs as today, the takeover unread;
- a records worktree with no upstream: `ReadUpstreamTakeover` reads not found;
- a prime records push whose rejection comes from another run's batten commit with no entry: replays and lands.
The listed test callers of `CoalescePushRequest` and `DetachedPush` pass a zero `MachineNames`.

**ImpactSummary:** Re-signs three fabricengine entries whose only production caller outside the package is `lym fabric push`'s detached child; the takeover read runs only on a rejected records push with names and a slug, so every unrejected push is unchanged.
