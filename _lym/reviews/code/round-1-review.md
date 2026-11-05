---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/gitrepo/pushdestination.go:19
    summary: the reflog subject "update by push" is declared twice, once as trackingReflogMessage where gitrepo now writes it and once as pushReflogSubject where fabricengine reads it, against PATTERN-named-constants
    fix: In internal/gitrepo/pushdestination.go replace the unexported const trackingReflogMessage with an exported const PushReflogSubject = "update by push", documented as the reflog subject of a remote-tracking ref a recorded push moves, and use it in both update-ref calls of moveTrackingRef. In internal/fabricengine/reflog.go delete the const pushReflogSubject, make trackingReflog's strings.HasPrefix test read gitrepo.PushReflogSubject, and reword the comment of that constant's former site and of trackingReflog so they say lym's own recorded push writes the subject, not git. Run a search for the literal "update by push" in non-test Go files and leave exactly one declaration.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/gitrepo/push.go:268
    summary: a failed local record step after a landed transfer is reported as a failed push, so DeleteRemoteBranch returns deleted true with an error that destroy.go reads as no deletion and records nothing, and a default push lands a failed-push record for a push origin holds
    fix: In internal/gitrepo/push.go make a recordPush failure after a successful transfer non-fatal in every caller (pushPlainWithRetry, DeleteRemoteBranch, DeleteRemoteBranchLeased, UpdateRemoteBranchLeased) - log it with logger.Warn naming the branch and tracking ref and return the success result (nil, or true, nil), since the next fetch corrects the tracking ref, the same reasoning the record's own design decision gives for a plain update-ref over a compare-and-swap. Delete the sentence "a failure of that local step alone returns deleted true with the error" from DeleteRemoteBranch's doc comment and state in recordPush's doc comment that its callers log its failure and never return it. Keep recordPush returning the error so the callers can log it.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/gitrepo/remoterefslock_integration_test.go:239
    summary: pushUnderHeldLock asserts at once, right after the bare remote shows the commit, that the push has not returned and the tracking ref is unchanged, so an implementation that wrote the tracking ref without the lock still passes while its unlocked write races that check
    fix: In pushUnderHeldLock replace the immediate non-blocking select on done (the one with a default branch) after the polling loop with a select on done against time.After(remoteRefsHeldWindow), failing when done closes first, so every caller's later check that the tracking ref and upstream are unchanged runs only after a push that ignored the lock has had time to write them. Keep the failure message.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gitrepo/pushdestination.go:43
    summary: the default push's remote choice (branch pushRemote, then remote.pushDefault, then the upstream remote, then origin) is new code with no test of the pushRemote, pushDefault or upstream-remote branches
    fix: Add rows to a table-driven test in internal/gitrepo (a new push_destination_integration_test.go beside remoterefslock_integration_test.go, tiergate 1, parallel) that give a clone two bare remotes and run PushRebaseFree with, in turn, branch.<name>.pushRemote set, remote.pushDefault set, only the branch's upstream remote set, and none set; each row asserts which bare remote received the branch and which tracking ref moved. Reuse publishedClone and branchSHA.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gitrepo/pushdestination.go:58
    summary: the no-record paths of recordPush have no test, namely a remote argument that names no configured remote (pushed to as a URL or path) and a configured remote whose fetch refspecs map the branch nowhere
    fix: In the table of the test added for F4, add two rows - PushBranch with a bare remote's filesystem path as the remote argument, asserting the push lands and no refs/remotes entry is created, and PushBranch to a remote whose fetch refspec maps only another branch, asserting the push lands and the repository's tracking refs are unchanged.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gitrepo/remoterefslock.go:24
    summary: comments the diff outdates were left, one saying a lock holder sees no push land in between, one saying an unreachable origin blocks every push, and one crediting push.autoSetupRemote for the first-push upstream
    fix: In internal/gitrepo/remoterefslock.go line 24 reword the sentence to say a holder that reads origin and then judges the tracking refs sees no fetch and no push's tracking-ref write in between. In internal/fabricengine/originreconcile.go lines 785-786 reword "blocks every fetch and push of both repositories" to say it blocks every fetch and every push's tracking-ref write of both repositories. In internal/boardengine/boardtest/sync_test.go lines 27-29 replace "otherwise do via push.autoSetupRemote" with wording that says the first real Sync's push would set the upstream.
---

### [BLOCKING] The reflog subject is declared twice

**Location:** internal/gitrepo/pushdestination.go:19, internal/fabricengine/reflog.go:16

**Issue:** `PATTERN-named-constants` asks for one named constant, declared once and referenced everywhere, for a name that can go stale. Git used to write the subject "update by push", and `fabricengine`'s `trackingReflog` reads it to find the last push boundary. The diff makes `gitrepo` write the subject itself with its own constant, so writer and reader now hold two independent copies of one contract. A change to either one silently breaks rewind and unpushed classification. `fabricengine` already imports `gitrepo`, so a single exported constant serves both sides.

### [MEDIUM] A failed record step after a landed transfer reports the push as failed

**Location:** internal/gitrepo/push.go:268 (also push.go:171, push.go:298, push.go:325), internal/fabricengine/destroy.go:1523-1527

**Issue:** Once the transfer has landed, `recordPush` can still fail on the lock, `update-ref`, `IsAncestor` or `git config`, and every caller returns that error. `DeleteRemoteBranch` returns `deleted == true` with the error, and `deleteRemoteBranch` records `KindRemoteBranchDeleted` only when the error is nil, so a landed remote delete goes unrecorded and a retry reads "remote ref does not exist". The leased calls return a bare error after the remote changed, which `updateRemoteBranch` words as "moving the remote task branch ... failed". A default push reaches `pushClassified` as an error no class names and leaves a failed-push record and creel notice for a push origin holds. The record's own design decision keeps the tracking-ref write a plain `update-ref` so that a failure cannot report a landed push as failed, and this path breaks the same reasoning.

### [MEDIUM] The held-lock test checks too early to catch an unlocked tracking-ref write

**Location:** internal/gitrepo/remoterefslock_integration_test.go:239

**Issue:** `pushUnderHeldLock` leaves its poll loop as soon as the bare remote shows the pushed commit, then checks once that the push has not returned, and its callers check once that the tracking ref and upstream are unchanged. A `recordPush` that skipped the lock needs only a few local git calls after the transfer, so it is still running at that moment and the checks pass. The fetch-and-pull test in the same file waits `remoteRefsHeldWindow` before it accepts that a call is blocked, and this helper needs the same wait.

### [MEDIUM] The default push's remote choice has no test

**Location:** internal/gitrepo/pushdestination.go:43

**Issue:** `resolvePushDestination` picks the remote from `branch.<name>.pushRemote`, `remote.pushDefault`, the branch's upstream remote and `origin`, in that order. The tests reach only the `origin` default of a first push and the upstream remote of a clone. A wrong order pushes the branch to the wrong remote and nothing fails.

### [LOW] The no-record paths of recordPush are untested

**Location:** internal/gitrepo/pushdestination.go:58

**Issue:** A remote argument naming no configured remote is pushed to as a URL or path and records nothing, and a remote whose fetch refspecs map the branch nowhere gives an empty tracking ref and takes no lock. No test drives either path, so a regression would write a bogus ref or take the lock needlessly.

### [LOW] Comments the diff outdates were left

**Location:** internal/gitrepo/remoterefslock.go:24, internal/fabricengine/originreconcile.go:785, internal/boardengine/boardtest/sync_test.go:28

**Issue:** `HoldRemoteRefs` still says a holder sees no push land in between, though a push transfer now lands and only its tracking-ref write is excluded. `judgeAgainstOrigin` says an unreachable origin blocks every fetch and push of both repositories, though only fetches and the push's record step wait. The board sync fixture says the first real Sync sets its upstream via `push.autoSetupRemote`, which no lym code sets any more.

## Rejected

- Fork code-tests, finding on the `deleted true, err` contract having no test: removed with F2, whose fix drops that contract.
- Fork code-tests, finding on the detached-HEAD early return and the same-named default push having no row: the early return passes a plain error through, which needs no row of its own, and the same-named default push is exercised by the `PushRebaseFree` row of the held-lock test and the first step of `TestPush`.
- Fork code-tests, finding that no row lands a push during a reconcile: the diff changes only comments of the reconcile, and `ReconcileOrigin` still holds both locks from its ls-remote pair to its last verdict as before; static prose is exempt.
- Fork code-correctness, finding that go-git's `repo.Config()` misses a remote defined only in global or included config: lym's repositories are clones and worktrees whose remotes sit in their own config file, and the existing `Upstream` read makes the same assumption.
- Fork code-correctness, finding that no "update by push" entry is written when a fetch already moved the ref: with the old lock a fetch that ran first left git's push the same no-op, so the reflog reads as it did before.
- Fork code-affected-set, finding that `toolfloor/doc.go` lists `push.autoSetupRemote` as a feature found: the gitexec and landing test fixtures still run `-c push.autoSetupRemote=true`, and that document lists keys tests set, so the entry stays true.
