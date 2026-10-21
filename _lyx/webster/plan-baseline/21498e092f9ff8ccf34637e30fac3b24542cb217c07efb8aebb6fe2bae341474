# Card 18 — reconcile-seams

**Create:**
- `plan:internal/pairteardown#SeamOptions` -> `type SeamOptions struct`
- `plan:internal/pairteardown#HubSeamOptions` -> `func HubSeamOptions(prime *lyxcwd.Location) SeamOptions`
- `plan:internal/pairteardown#Teardown.OriginSeams` -> `func (t *Teardown) OriginSeams(opts SeamOptions) fabricengine.OriginSeams`
- `plan:internal/pairteardown#TestTeardown_OriginSeams` -> `func TestTeardown_OriginSeams(t *testing.T)`
- `plan:internal/pairteardown#TestHubSeamOptions` -> `func TestHubSeamOptions(t *testing.T)`

**Uses:**
- `internal/pairteardown#Teardown.probeQuiet`
- `internal/fabricengine#OriginSeams`
- `internal/fabricengine#EndedTeardown`

**Intent:** D1's injected seams, built in one place every call site's CLI package uses, since `pairteardown` already imports both `fabricengine` and `reedengine`.
`Teardown.OriginSeams` returns a `fabricengine.OriginSeams`:
- `LivePair(slug)` is the negation of the quiet rule's probe (run lock, loop lock, driver strand), except that a slug a live hub pause entry covers reads not live, since its driver is interrupted; `SeamOptions.HubPaused` is a `func(slug string) (bool, error)` that `LivePair` calls each time it is called, so a pause lifted while the reconcile's ls-remote waited at the network gate reads lifted and the resumed driver reads live;
- `PrimeParentLockFree` is `SeamOptions.PrimeParentLockFree`;
- `Teardown(EndedTeardown)` runs `Run` with `Local` true, a zero `QuietWait` and `RefuseWhenBusy`, fills the report's `MutationRecord` from the result's `Removal` record on every return, and reports a `fabricengine.ErrPairBusy` or `ErrDriverBusy` as the report's busy flag; the force case, when `SeamOptions.WakeHeld(slug)` reports a live wake-written pair pause entry of the slug or `EarlierSnapshot` is set, instead runs it without `RefuseWhenBusy`, with `Force` and `Snapshot` set and `AfterSnapshot` lifting that entry through `SeamOptions.LiftWakeHold`.
`HubSeamOptions` fills `HubPaused` with a read of the pause record at `hubgeom.PauseGeometry` made at each call, never a value read when the seams are built, and `PrimeParentLockFree` from one `parentlock` store listing of the prime branch, and leaves `WakeHeld` and `LiftWakeHold` nil, which read as no wake hold; `lyx cleanup` fills those two (card 20).
Bound: the force case ends a busy driver only for an ended entry holding a wake-written pair pause entry, or the snapshot tag an earlier run of the case left; every other teardown refuses a busy driver.

`TestTeardown_OriginSeams` (untagged, `Teardown` fakes) rows: a quiet pair is not live and a busy one is; a busy pair under a live hub pause entry is not live, and reads live once the fake `HubPaused` reports the entry lifted, with no new seams built; the teardown seam passes `Local`, `RefuseWhenBusy` and a zero wait, and a busy driver comes back as the busy flag with nothing removed; a fake removal's recorded mutation comes back in the report's `MutationRecord`; with `WakeHeld` true it passes `Force` and `Snapshot` without `RefuseWhenBusy` and lifts the hold after the snapshot; with `EarlierSnapshot` and no hold it does the same with nothing to lift.
`TestHubSeamOptions` (integration, package `pairteardown`, a prime resolved with `lyxcwd.ResolveWorktree` over a `git init` repo in a temp hub as `TestProbeQuietSeesTheGoDriverLock` builds its task worktree) rows: with no pause entry and no parent lock record the pair's slug reads not paused and the prime's parent lock free; a live hub entry written through `pauserecord.Update` at `hubgeom.PauseGeometry(prime)`'s record path makes `HubPaused` true for the slug, and the same entry lifted after `HubSeamOptions` returned makes the same `HubPaused` read false; a parent lock record of the prime branch naming the live test process makes `PrimeParentLockFree` false.

**Commit:** 18: pairteardown: the origin reconcile's seams
