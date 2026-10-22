# Card 7 — Merge-In, Finalize and Publish tell a hold that a parent lock exempts from slug entries

**Create:**
- `plan:internal/parentlock#Store.HeldBy` -> `func (s *Store) HeldBy(slug string) (bool, error)`
- `plan:internal/parentlock#TestStore_HeldBy` -> `func TestStore_HeldBy(t *testing.T)`
- `plan:internal/loomcli#holdsParentLock` -> `func holdsParentLock(store *parentlock.Store, slug string) func() bool`
- `plan:internal/loomcli#TestHoldsParentLock` -> `func TestHoldsParentLock(t *testing.T)`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed#verifyGate.checkPublishVerify`
- `internal/landingshed#TestVerifyGate_CheckMapsVerifyResult`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#intentionallyNil`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`
- `internal/tendercli#InstallNetworkGate`

**Uses:**
- `plan:internal/verifytree#NewPauseHold`
- `plan:internal/verifytree#PauseHold`
- `internal/parentlock#Store.List`

**Intent:**
D5 for landingshed's Merge-In, Finalize and Publish verify site, whose run holds the parent's lock while it verifies.
`landingshed.Deps` gains the field `VerifyHold *verifytree.PauseHold`, documented as the told pause hold of the post-merge verifies, nil holding nothing.
`verifyGate` gains a `hold` field that `newVerifyGate` copies from `deps.VerifyHold`, and `check` and `checkPublishVerify` pass it as `Site.Hold` on their verify calls.

`parentlock.Store.HeldBy` reports whether a live lock record with no read error is owned by `slug`, over `List`, and returns `List`'s error; it is the rule `tendercli.InstallNetworkGate`'s `HoldsParentLock` closure applies today, which now calls it and keeps its own Warn and not-held reading of an error, so the network gate and the verify hold share one rule.
`landingDeps` sets `VerifyHold: verifytree.NewPauseHold(runner.PauseGeometry(), holdsParentLock(lock.Store, l.WorktreeName))` from the runner and lock deps it is already handed, so it stays an assembly with no I/O and no new parameter.
`holdsParentLock` returns the function that calls `store.HeldBy(slug)`: a nil store reads not held, and an error reads not held with one Warn.
While it reports true, the hold reads hub entries only, so a slug entry never holds or stops a verify the run takes under the parent's lock.

The new `Deps` pointer meets every `NilSeams` check over an `Env`: loomcli's `wire` and battencli's env both leave `Env.Landing` to their own assembly, so `Landing.VerifyHold` joins loomcli's `intentionallyNil` with the reason the other `Landing.*` seams carry and battencli's with `loom-only seam, batten lands no branch`, and `envkit.LandingDeps` fills it with `verifytree.NewPauseHold(pauserecord.Geometry{}, nil)`, so `TestFullEnv_FillsEverySeam` stays green.

`TestVerifyGate_CheckMapsVerifyResult`'s fake verify records the site it is called with, and the test asserts the site carries the gate's hold for both `check` and `checkPublishVerify`.
`TestLandingDeps_EveryFieldPopulated` already fails on a zero `Deps` field, and `NewPauseHold` never returns nil, so it pins that `landingDeps` fills the hold; it also asserts the hold's exemption is wired, by building the deps over a runner told a temp record holding a slug entry of the worktree's slug past its grace end and a lock store holding a live lock owned by that slug, and reading `VerifyHold.PastGrace()` false, and true again with the lock released.
`TestStore_HeldBy` is a table over a temp store: a live lock owned by the slug reads held, a live lock of another slug and a dead owner's lock read not held, and a store whose directory is a file returns the list error.
`TestHoldsParentLock` covers the wrapper's own two branches: a nil store reads not held, and a store whose directory is a file reads not held across two calls with one Warn.

**ImpactSummary:** one new `Deps` field read by landingshed's verify gate at its two call sites, filled by loomcli's `landingDeps` from values it already takes and by envkit's fixture; one parent-lock reading moved into `parentlock` and shared with tender's network gate, whose behaviour is unchanged.
