---
verdict: BLOCKING
findings:
  - id: F16
    carries: F2
    severity: BLOCKING
    class: consistency
    location: _lym/plan/10-teardown-pushes-first.md:40
    summary: "Card 10 adds Teardown.Push and Teardown.Level to both envkit's nilLegal and loomcli's intentionallyNil, but the two lists exclude each other: envkit.NilSeams skips every nilLegal path, so loomcli's TestWire_EverySeamFilled never sees them nil and its second loop fails with 'intentionallyNil lists c.env.Teardown.Push but wire() now fills it; remove the entry'. Card 10 cannot pass its own gate."
    fix: "In card 10, remove `internal/loomcli#intentionallyNil` from the **Edit:** list and drop the clause 'and loomcli's `intentionallyNil` gains both as batten-only seams battencli fills' from the TeardownDeps bullet. Keep `internal/testkit/envkit#nilLegal` gaining Teardown.Push and Teardown.Level, which covers loomcli's wire, battencli's wire and FullEnv alike. Checked with lym loom validate-plan --overlay together with F17's fix: no new blocking finding."
  - id: F17
    severity: BLOCKING
    class: design
    location: _lym/plan/01-fabric-owed-unrecorded.md:48
    summary: "Card 1's checkOrigin bullet deletes the local tracking ref of a branch gone from origin with its entry ended, but plans no way to delete only the ref. The one gitrepo delete, Repo.DropRemoteTracking (internal/gitrepo/push.go), also unsets branch.<branch>.remote and .merge. With the upstream unset the branch reads as no-upstream: card 2's RunCommitsAhead reads it ahead and TipsOnOrigin reads it not held, so card 10's push-then-Level loop and the dobby restart never end, and the next push (UpstreamSHA returns ErrNoUpstream, checkOrigin proceeds) recreates the deleted branch on origin. The ref delete also writes a remote-tracking ref, so PATTERN-gitrepo-client-boundary needs it in gitrepo under the remote-refs lock, mapped through the remote's fetch refspecs as in pushdestination.go; no card targets gitrepo."
    fix: "In card 1 add the **Create:** target `plan:internal/gitrepo#Repo.DeleteRemoteTrackingRef` -> `func (r *Repo) DeleteRemoteTrackingRef(remote, branch string, ns RefNamespace) error` and `internal/gitrepo/doc.go` under **Uses:** (the doc-go-unlisted check requires it). In the checkOrigin Intent bullet say checkOrigin deletes the ref through it: it removes only the remote-tracking ref the remote's fetch refspecs map the branch's origin ref under ns to, in one hold of the remote-refs lock, touches no config, and the branch keeps its upstream setting so it reads as upstream-set with a pruned ref; DropRemoteTracking is not used because it also unsets the upstream. In the two ended-branch rows of TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide add the assertion that the tracking ref is gone while branch.<branch>.merge stays set, which reaches the new function through the public surface without a separate gitrepo test. Checked with lym loom validate-plan --overlay: the card with these changes, alone and together with F16's fix, reports no blocking finding the baseline lacks."
prior:
  - id: F1
    status: resolved
    answer: "Card 3's retryWeftPush bullet runs primeTakeovers on every prime records push whatever runSlug is, adds the slug-naming prime row to TestRejectedRecordsPush_ReadsTheTakeover, and card 4 states PushRun's TakeoversError branch serves every prime push."
  - id: F2
    status: open
    answer: "Card 10 now edits both lists, but the two edits contradict each other (see F16), so TestWire_EverySeamFilled still fails."
  - id: F3
    status: resolved
    answer: "Card 8 starts the pair's pusher only inside the !OwnedElsewhere && PairPresent condition, and HandOverOwnedElsewhere does push the run's owed commits before it hands over (internal/pairteardown/handover.go)."
  - id: F4
    status: resolved
    answer: "Card 6's pusherCmd bullet gives the resolve failure a trailing way forward clause and the TestResolvePersistentPreRun_SkipsSlugGroups row asserts it."
  - id: F5
    status: resolved
    answer: "The fixed-differently route is accepted: a local read cannot tell a branch gone from origin from one still there, so deleting the stale tracking ref at the push is the right place. The remaining gap, how that delete is made without unsetting the upstream, is raised as F17."
  - id: F6
    status: resolved
    answer: "Card 3's TestHandOverTips rows set PrimeRecords with Pair nil, and HandOverTips guards its pair archive on req.Pair != nil, so the prime archive is the first step that can fail."
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
    answer: "Accepted on the same ground as F5; the delete mechanism is raised as F17."
  - id: F14
    status: resolved
  - id: F15
    status: resolved
---

### [BLOCKING] Card 10 lists the same seams in two lists that exclude each other

**Location:** `_lym/plan/10-teardown-pushes-first.md:40`
**Issue:** `envkit.NilSeams` (`internal/testkit/envkit/envkit.go`, `walkNil`) skips every path in `nilLegal`.
Once card 10 adds `Teardown.Push` and `Teardown.Level` there, `NilSeams(c.env)` in loomcli's `TestWire_EverySeamFilled` never reports them.
That test's second loop (`internal/loomcli/wiring_seams_test.go`) fails for each `intentionallyNil` key that is absent from `nils`,
so the same card adding both seams to `intentionallyNil` breaks it.
The nil-legal entry alone satisfies loomcli's wire, which fills no teardown seam, battencli's wire, which fills both, and `FullEnv`.
This carries F2.

### [BLOCKING] Card 1 plans no way to delete only a tracking ref, and the existing delete unsets the upstream

**Location:** `_lym/plan/01-fabric-owed-unrecorded.md:48`
**Issue:** The round-1 answer to F5 and F13 rests on `checkOrigin` deleting the stale tracking ref of an ended branch, so card 2's reads see it level and held.
`gitrepo.Repo.DropRemoteTracking` is the only exported ref delete and it also unsets `branch.<branch>.remote` and `.merge`.
A branch left with no upstream is the case card 2 reads as ahead and not held, so the loop F5 named still never ends.
It also lets the next push through `checkOrigin`'s `ErrNoUpstream` branch, which recreates a branch origin deleted.
The tracking ref lives under the refspec-mapped name that `resolvePushDestination` computes, and writing it belongs in `gitrepo` under the remote-refs lock (`PATTERN-gitrepo-client-boundary`).
No card targets `gitrepo`, so an implementer has no planned way to meet the bullet.
The focus question on card 10's push-then-Level loop is settled by F17: it ends only once the delete keeps the upstream.
