# Card 18 — reconcile-seams

**Create:**
- `plan:internal/pairteardown#SeamOptions` -> `type SeamOptions struct`
- `plan:internal/pairteardown#HubSeamOptions` -> `func HubSeamOptions(prime *lyxcwd.Location) SeamOptions`
- `plan:internal/pairteardown#Teardown.OriginSeams` -> `func (t *Teardown) OriginSeams(opts SeamOptions) fabricengine.OriginSeams`
- `plan:internal/pairteardown#TestTeardown_OriginSeams` -> `func TestTeardown_OriginSeams(t *testing.T)`
- `plan:internal/pairteardown#TestHubSeamOptions` -> `func TestHubSeamOptions(t *testing.T)`

**Uses:**
- `internal/pairteardown#Teardown.probeQuiet`
- `plan:internal/fabricengine#OriginSeams`
- `plan:internal/fabricengine#EndedTeardown`

**Intent:** D1's injected seams, built in one place every call site's CLI package uses, since `pairteardown` already imports both `fabricengine` and `reedengine`.
`Teardown.OriginSeams` returns a `fabricengine.OriginSeams`:
- `LivePair(slug)` is the negation of the quiet rule's probe (run lock, loop lock, driver strand), except that a slug a live hub pause entry covers (`SeamOptions.HubPaused`) reads not live, since its driver is interrupted;
- `PrimeParentLockFree` is `SeamOptions.PrimeParentLockFree`;
- `Teardown(EndedTeardown)` runs `Run` with `Local` true, a zero `QuietWait` and `RefuseWhenBusy`, and reports a `fabricengine.ErrPairBusy` or `ErrDriverBusy` as the report's busy flag; the force case, when `SeamOptions.WakeHeld(slug)` reports a live wake-written pair pause entry of the slug or `EarlierSnapshot` is set, instead runs it without `RefuseWhenBusy`, with `Force` and `Snapshot` set and `AfterSnapshot` lifting that entry through `SeamOptions.LiftWakeHold`.
`HubSeamOptions` fills `HubPaused` from the pause record at `hubgeom.PauseGeometry` and `PrimeParentLockFree` from one `parentlock` store listing of the prime branch, and leaves `WakeHeld` and `LiftWakeHold` nil, which read as no wake hold; `lyx cleanup` fills those two (card 20).
Bound: the force case ends a busy driver only for an ended entry holding a wake-written pair pause entry, or the snapshot tag an earlier run of the case left; every other teardown refuses a busy driver.

`TestTeardown_OriginSeams` (untagged, `Teardown` fakes) rows: a quiet pair is not live and a busy one is; a busy pair under a live hub pause entry is not live; the teardown seam passes `Local`, `RefuseWhenBusy` and a zero wait, and a busy driver comes back as the busy flag with nothing removed; with `WakeHeld` true it passes `Force` and `Snapshot` without `RefuseWhenBusy` and lifts the hold after the snapshot; with `EarlierSnapshot` and no hold it does the same with nothing to lift.
`TestHubSeamOptions` (integration, package `pairteardown`, a prime resolved with `lyxcwd.ResolveWorktree` over a `git init` repo in a temp hub as `TestProbeQuietSeesTheGoDriverLock` builds its task worktree) rows: with no pause entry and no parent lock record the pair's slug reads not paused and the prime's parent lock free; a live hub entry written through `pauserecord.Update` at `hubgeom.PauseGeometry(prime)`'s record path makes `HubPaused` true for the slug; a parent lock record of the prime branch naming the live test process makes `PrimeParentLockFree` false.

**Commit:** 18: pairteardown: the origin reconcile's seams
