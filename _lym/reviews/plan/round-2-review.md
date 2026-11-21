---
verdict: APPROVED
findings:
  - id: F16
    severity: MEDIUM
    class: design
    location: _lym/plan/00-overview.md:55
    carries: F8
    summary: "The coverage-of-namespace-sites decision still leaves one-repository sites that no planned row reaches: probeLiveLocalWeft, readSideOwnerArchives, Fabric.PushWeft, and the fetch of targetWeftTip, which card 6's Checkout row never reaches because its target records branch exists nowhere. An identity-namespace read, fetch or push left at any of them passes every planned row. The overview's claim that card 6's Checkout row reaches targetWeftTip holds only for the branch-absent exit."
    fix: "Edit 00-overview.md's coverage-of-namespace-sites decision: delete the clause that card 6's Topology.Checkout row reaches checkout's targetWeftTip, and delete the sentence that no one-repository row reaches Fabric.PushWeft, probeLiveLocalWeft or readSideOwnerArchives; name the sites card 7 reaches instead. In 07-pair-lifecycle-namespace.md add four bullets to TestOneRepositorySecondMachine_AdoptPullAndReplay, and add `internal/fabricengine#Topology.Checkout`, `internal/fabricengine#ArchiveOwnerTips`, `internal/fabricengine#Topology.ReadOwnerArchives` and `internal/fabricengine#Fabric.PushWeft` to its Uses. Bullet one, live local branch: with the second machine's records branch present locally and origin's refs/lym/heads/<branch>-weft ahead of it, Topology.Add of the slug reaches probeLiveLocalWeft and reports a fast-forward to origin's tip. Bullet two, checkout: the second machine pushes a pair whose records branch the first hub holds nowhere locally; the first hub's Topology.Checkout of the prime to that branch reaches targetWeftTip, which fetches refs/lym/heads/<branch>-weft into refs/remotes/origin/<branch>-weft, and the records worktree ends on that branch with the second machine's commits. Bullet three, records push: Fabric.PushWeft on a hub holding an unpushed records commit lands it at refs/lym/heads/<branch>-weft and adds no ref under refs/heads/. Bullet four, owner archives: ArchiveOwnerTips on the first hub for a pair holding an unpushed records commit pushes its owner archive tag under refs/lym/tags/, and Topology.ReadOwnerArchives on the second machine lists it on the records side and fetches it as a local tag. Checked with lym loom validate-plan --overlay: the card 7 text with these Uses lines and bullets reports only the two baseline pattern-changes-mismatch findings."
prior:
  - id: F1
    status: resolved
    answer: Card 4's pushCloneSeed now lives in destroy.go as an executor of the destruction gate. It runs checkPathRequest over the hub path with ownedFreshlyCreatedPath, names the two refs it may push or restore, declares dirtinessNA with a reason and force false, and refuses before any push.
  - id: F2
    status: resolved
    answer: "The listed added entries \"PATTERN change: lym's own records-namespace branch names are named constants\" and \"Declare the PATTERN-code-conventions change for lym-owned branch names\" settle it. The overview declares the change, cards 2 and 4 hold one fabric constant each, and card 8 writes the Branch rule."
  - id: F3
    status: resolved
    answer: Card 2's TestDeleteRemoteBranch rows now use x-records and y/x-records, and no planned row in a package outside the fabric vocabulary owner set spells the records side token.
  - id: F4
    status: resolved
    answer: The unseeded-board refusal sits in pushClassified for side bolt before the upstream read, so Bolt.Push, PushRecorded, Sync and a reconcile all meet it. It writes no failed-push record, and card 5's unseeded-board row reaches it.
  - id: F5
    status: resolved
    answer: Card 4's pushCloneSeed sets branch.main.remote and branch.main.merge after the board push lands, and card 5's seed test asserts the merge key and HasUnpushed false.
  - id: F6
    status: resolved
    answer: Card 5's FailedSeedPushLeavesNothing test now has the abandoned-seed and abandoned-two-URL rows, and card 4 no longer claims no test reaches AbandonSeededClone.
  - id: F7
    status: resolved
    answer: Card 7's one-repository row of the never-pushed cleanup test first creates a same-named code branch on the bare, so a tip read that ignores the namespace deletes it and fails the row.
  - id: F8
    status: open
    answer: Carried as F16. The added rows cover add with the leftover probe, push, both replays, pull, hand-over, remove, cleanup, reconcile and merge, but not probeLiveLocalWeft, readSideOwnerArchives, Fabric.PushWeft or the fetch inside targetWeftTip. The two-repository suites do not reach those sites in a one-repository hub, so the fixer's reason does not close the gap.
  - id: F9
    status: resolved
    answer: Each new production function now lands with its first caller in the same card. The test seam setter sits in export_test.go, and the test-support packages are roots of the guard.
  - id: F10
    status: resolved
    answer: Card 4's join fetch goes through gitrepo.New(dir).Fetch() under the remote-refs lock.
  - id: F11
    status: resolved
    answer: Card 4 states that recordedAnchorOnWeft skips its HEAD fallback under a non-identity namespace, and card 5's reset row covers a bare with no records branch and a missing subpath.
  - id: F12
    status: resolved
    answer: I reproduced the fixer's probe with git 2.53.0 in a scratch bare holding refs/lym/heads/main.lock. A push under a lease on absence prints "reference already exists", so card 4's transient classification and card 5's transient row hold.
  - id: F13
    status: resolved
    answer: Card 3 drops hasWeftEvidence and RecordsBranchFile from Edit and states that RecordsBranchFile needs no change.
  - id: F14
    status: resolved
    answer: Card 2 names the tier0-only file for TestRefNamespace and remote_integration_test.go, which exists, for TestRemoteFetchRefspecs.
  - id: F15
    status: resolved
    answer: Card 4's Seed bullet no longer carries the clause about an origin whose HEAD names no branch.
---

### [MEDIUM] One-repository namespace sites that no planned row reaches

**Location:** `_lym/plan/00-overview.md:55`, with the rows to add in `_lym/plan/07-pair-lifecycle-namespace.md`

**Issue:** This carries F8, narrowed to what remains.
Card 7 now reaches add with its leftover probe, push, both replays, pull, hand-over, remove, cleanup, reconcile, remote cleanup and merge.
Four sites that cards 2 and 3 change still reach no row over a one-repository hub:

- `probeLiveLocalWeft` runs only when the records branch exists locally and origin's tip is not an ancestor of it.
  No row creates that state.
- `readSideOwnerArchives` is reached through `Topology.ReadOwnerArchives`, which no row calls.
  Its fetch of `refs/tags/<tag>` must become the namespaced tag.
- `Fabric.PushWeft` pushes through `PushCoalesced`, which takes the namespace.
  It has no production caller, and no row calls it either.
- `targetWeftTip` fetches only when the target records branch is absent locally and present on origin.
  Card 6's Checkout row moves the prime to a new branch, so `remoteHeadTip` answers empty and the function returns before its fetch.
  The overview says that row reaches `targetWeftTip`, which is true of the function and false of its namespaced fetch.

The overview's claim that the two-repository suites keep the coverage does not hold, because those suites run the identity namespace.
A push or fetch left at `refs/heads/` or `refs/tags/` in any of these sites would pass every planned row.

## Focus verdicts

- Destructive remote primitives: `pushCloneSeed` and its restore sit in `destroy.go` and run the gate's four checks before each push.
  Card 2's probing plain delete adds no destructive call.
  Not a defect.
- Branch literals: `OneRepositoryBoardBranch` in card 2 and `legacyBoltBranch` in card 4 are one fabric constant each, as the declared `PATTERN-code-conventions` text words it.
  No other card spells a branch.
  Not a defect.
- Vocabulary: the planned `gitrepo` rows use `x-records`, and `hubforge`, `fabriccli` and `fabricengine_test` are in the owner set.
  Not a defect.
- Reachability: every new production function lands in the card that holds its first caller.
  `SetSeedPushHookForTest` is a `_test.go` helper.
  Webster groups cards by cost, so no card ordering can promise a batch boundary, and each card passing alone is the safe shape.
  Not a defect.
- One-repository seed: every board push goes through `pushClassified`, which refuses an unseeded board before any push, and the seed sets the upstream.
  An interrupted clone and a later reconcile stay bounded.
  Not a defect.
- Refusals: each clone and pair refusal has a named row whose input no upstream check rejects.
  The `fabriccli` row for a second URL reaches the offline refusal in `CloneHub`, which runs before any network call.
  Not a defect.
- Card 7's reach: the finding above.

## Rejected

- `pattern-changes-mismatch`: the overview declares `PATTERN-single-repo-hub` as `changes` while the decision record declares `adds`.
  The plan gate reports it as blocking in both the baseline and the overlay run, so it is not raised here.
- The way-forward text at `remoteteardown.go` tells an operator to run `git push origin refs/tags/<tag>`, which in a one-repository records store would push to the code tag namespace.
  It is message text, not a ref fabric sends, and an incremental round raises only a blocking finding on a changed hunk, so it is not raised.
