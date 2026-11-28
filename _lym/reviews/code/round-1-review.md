---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/parentalign.go:265
    summary: The owed-only check runs with no lock, and the local tip it licenses is read later under the lock with no re-check, so a non-landing commit made on the parent in between is merged into the task branch.
    fix: "In parentMergeTip, once guard.ensure holds the lock and ReadParentTips(false) has read the local tip, and only on the takeOwedLocal path for a parent first read as ParentAhead, call deps.ParentAheadByOwed again. When it answers false, return the upstream tip read before the lock instead of tips.Local, and wrap a failed read with the same wording as the lock-free read. Add a test row: ParentAheadByOwed answers true on the first call and false on the second, and the lock-free pass merges the upstream tip."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_unlocked_test.go:75
    summary: Two fallbacks in Finalize's lock-free pass have no test that fails when they break. A parent ahead of its upstream that is not owed-only, and an ahead parent with a nil ParentAheadByOwed, should both merge the upstream tip, and returning tips.Local for every ahead parent passes every test.
    fix: "In TestFinalize_GatesWithNoLockAndLocksToLand, or in TestFinalize_LockFreePassStops since the locked sync then refuses, add a row where ReadParentTips answers ParentAhead with Upstream u0 and ParentAheadByOwed answers false. Add a second row where ParentAheadByOwed is nil. Assert in each row that the first merged parent tip is u0."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/parentalign_test.go:334
    summary: No test pins that Parent-Align keeps the upstream tip of an owed-ahead parent. newMergeInFixture leaves ParentAheadByOwed nil, so passing true at parentalign.go:147 would still pass every test.
    fix: "In TestMergeIn_MergeTip, add a row like the ahead row at parentalign_test.go:334, with tips Local p2, Upstream p1, ParentAhead. Before the call, set fx.m.deps.ParentAheadByOwed to a func returning true and nil, through a new owedAhead field on the row. Assert wantTip p1 and that the lock is never taken."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/finalize.go:380
    summary: For a landing that merges on GitHub, gateUnlocked now merges the parent's owed, unpushed landing tips into the task branch before the parent-push check in landUnderLock ends the call, which the GitHub path never did before.
    fix: "In gateUnlocked, pass takeOwedLocal as !fz.needsGitHubOnDisk() instead of true, so a landing known to need GitHub keeps merging the upstream tip while the parent owes a push. Extend the gateUnlocked doc line about owed landing tips to name the exception."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/landingshed/parentalign.go:229
    summary: The parentMergeTip doc still says that with an upstream it takes the upstream tip, never the local tip, so an unpushed local landing is never merged in. With takeOwedLocal set it now merges exactly such landings.
    fix: "In the parentMergeTip doc comment, change the sentence at parentalign.go:229-230 so it no longer says \"never the local tip\" unconditionally. For example: with an upstream and no push skip, it fetches and takes the upstream tip, so a commit on the local parent that origin lacks is not merged in, except the owed landing tips below."
---

### [MEDIUM] Owed-only check and local-tip read are not atomic

**Location:** internal/landingshed/parentalign.go:265

**Issue:** `parentMergeTip` reads `ReadParentTips(true)` and `ParentAheadByOwed` with no lock, at lines 241-257.
It then takes the lock and returns `tips.Local`, read at lines 265-274.
A landing in between is safe, because it merges and records under the lock (`finalize.go:555-559`), so the new tip is owed too.
An owed push that completes in between is also safe, because local then equals upstream.
A plain commit on the parent, such as an operator commit in the parent pair, is not safe: it is returned as the tip and merged into the task branch.
The locked pass's `syncParent` then refuses the parent as ahead (`finalize.go:722-731`), but the task branch already holds the unpushed commit.
It can then reach origin through the task branch, or land again after the operator reconciles the parent by dropping or rewriting it.
The old code never merged a local-only commit in the lock-free pass while the parent had an upstream.

### [MEDIUM] No test for the upstream fallbacks of an ahead parent in Finalize

**Location:** internal/landingshed/finalize_unlocked_test.go:75

**Issue:** The only `ParentAhead` rows that reach the lock-free pass are the new owed-true row and the failed-read row (`finalize_unlocked_test.go:91-96`, `:283-286`).
`finalize_test.go:1150` drives `syncParent` alone.
Two branches at `parentalign.go:248-257` therefore have no test: the `owed == false` branch and the nil-seam branch.
Both are behaviours the diff adds, so a regression that merges the local tip of any ahead parent goes unseen.

### [MEDIUM] No test pins Parent-Align's upstream tip for an owed-ahead parent

**Location:** internal/landingshed/parentalign_test.go:334

**Issue:** The ahead row of `TestMergeIn_MergeTip` runs with `ParentAheadByOwed` nil, so the `!takeOwedLocal` guard is never the branch that decides.
The summary's design decision, that Parent-Align merges only what origin holds, is unpinned.

### [LOW] A GitHub landing merges owed tips before its push check

**Location:** internal/landingshed/finalize.go:380

**Issue:** `Finalize.Call` runs `gateUnlocked` (`finalize.go:341`) before `landUnderLock`, and the GitHub push check `parentPushBlocks` runs only inside `landUnderLock` (`finalize.go:423-427`, `:453-457`).
With an owed parent and a pull-request landing, the lock-free pass now merges the owed landing tips into the task branch and gates them, then the call ends transient.
The old code merged the upstream tip there, and its locked merge ran only after the push check cleared.
If the owed push is later refused and the parent is reconciled by a rewrite, the resumed run pushes stale landing commits through `pushTask` (`finalize_pr.go:474`) into the pull request.
The lock-free gate is also spent on a tree that cannot land until origin holds those tips.

### [LOW] parentMergeTip doc contradicts the new branch

**Location:** internal/landingshed/parentalign.go:229

**Issue:** Lines 229-230 say "takes the upstream tip, never the local tip, so a landing another run made locally and has not pushed is never merged in", while lines 232-234 say it takes the local tip of an owed-ahead parent when `takeOwedLocal` is set.

## Rejected

- NIT (lens:code-affected-set) that the `Deps.ParentAheadByOwed` doc at `deps.go:180-186` omits its use by the lock-free pass: the doc states what the seam reports and stays true, and listing each consumer is the enumerated-consumer pattern the comment rules forbid.

## Notes on the focus entries

- Agreement of the two reads: `ReadParentTips` reads `refs/heads/<parent>` and its upstream in the hub's shared code repository, and `AheadByOnly` reads the parent pair's checked-out branch against its tracking ref after the same fetch. Both concern the same branch, repository and upstream, the same pairing `syncParent` already relies on. Not a defect.
- After the merge: the merge record's `MergedTip` and overlap use the local tip consistently (`finalize.go:1401-1422`). The locked pass sees `head == gatedTip`, merges `""` as already up to date and skips the verify. A local landing that merges owed tips into the task branch did so before too, under the lock. Only the pull-request path is new exposure (F4).
- Summary claims hold: other ahead parents merge the upstream tip and `syncParent` refuses them, and the before-the-fix behaviour of the new row matches the old code's lost-race path. The two-way door holds, since no record format, config meaning or outward push changes, and a blast radius of landing covers the diff.
