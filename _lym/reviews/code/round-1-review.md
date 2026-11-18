---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pushanchored.go:230
    summary: A records side still unpushed once the retry backoff is spent, and a transport failure of the post-push origin lookup, return errors IsPushOwed reads as not owed, so these transient states still fail the run at Seed-IDE.
    fix: In internal/fabricengine/pushanchored.go, make both transient outcomes read as owed pushes. When pushRecordsSide spends its backoff on errRecordsUnpushedPastPush, write a failed-push record of class PushFailureDeferred for the records side and branch through recordPushFailure (as teardownLockOwedError does), and return its *PushFailedError wrapping the not-landed error. In recordsSideNotLanded, when branchGoneOnOrigin fails with gitexec.IsTransportFailure, record and return a PushFailureTransport *PushFailedError the same way. Leave a non-transport lookup error and the ended message unclassified. Update the PushPairAnchored and recordsSideNotLanded doc comments and the doc.go records-side lines to say these outcomes are recorded as owed.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pushanchored_integration_test.go:523
    summary: The new retry has no test for its spent backoff or a failed post-push origin lookup, and the new subtest does not count its backoff sleeps.
    fix: In TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide, add a subtest whose records-bare post-receive hook commits to the prime records worktree on every push; pass countingBackoff(&sleeps, nil) and assert sleeps == len(gitrepo.PushBackoffDelays()), the error and failed-push record F1 settles on, and that IsPushOwed reports true. Add a row that makes the post-push ls-remote fail with a transport failure (for example a post-receive hook that repoints the prime records origin URL at a missing path after the first push) and assert the transport classification F1 settles on. In PrimeRecordsCommitDuringThePushIsPushedAgain, replace the no-op BackoffSleep with countingBackoff(&sleeps, nil) and assert sleeps == 1.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/fabricengine/fabric.go:82
    summary: The SyncOptions.BackoffSleep doc names only the wait before a missing branch or unreachable origin is recorded, but the field now also paces the records side's re-push.
    fix: Reword the doc comment on SyncOptions.BackoffSleep in internal/fabricengine/fabric.go to state what it does without listing its uses, for example "BackoffSleep sleeps one delay of the push backoff between a push's retries; nil means sleep."
    origin: lens:code-affected-set
---

### [MEDIUM] Transient not-landed outcomes still fail Seed-IDE

**Location:** internal/fabricengine/pushanchored.go:230

**Issue:** The board entry expects that a transient classification does not fail the run at Seed-IDE.
The transition seam retries a push only while `fabricengine.IsPushOwed` holds (`internal/statuscommit/statuscommit.go:124`, wired at `internal/battencli/commitstatus.go:174`),
and `IsPushOwed` holds only for a `*PushFailedError` of class transport, deferred or teardown-lock, or for `ErrPushLockBusy` (`pushanchored.go:44-56`).
Once the records retry spends its backoff (`attempt == len(delays)`), it returns a plain error wrapping `errRecordsUnpushedPastPush`, which is not owed, so a records branch that other writers keep committing to through the backoff still fails the run.
A transport failure of the new post-push `branchGoneOnOrigin` lookup (`pushanchored.go:332-335`) is returned unretried and unclassified, so it fails the run too,
whereas the pre-push lookup retries a transport failure over the backoff (`pushclassified.go:540-558`).
The single commit-during-push case the Seen describes is fixed; these residual transient paths are not.
The assistant reviewer's BLOCKING finding on the same defect is merged here; I hold it at MEDIUM because both paths need a transient state to outlast the bounded retry or to strike the one lookup right after a push that landed.

### [MEDIUM] The retry's failure outcomes are untested

**Location:** internal/fabricengine/pushanchored_integration_test.go:523

**Issue:** The new subtest's hook commits once, so the second push lands; nothing reaches the `attempt == len(delays)` exit or the failed-lookup return.
A search of fabricengine's tests finds no assertion on `errRecordsUnpushedPastPush`, its message, or "look up branch".
An off-by-one in the bound, a dropped bound, or a lost classification would pass the suite.
The subtest also passes a no-op `BackoffSleep` without counting calls, so the retry's use of `opts.BackoffSleep` and its sleep count are unpinned.
Merges the tests lens's three findings and the assistant reviewer's coverage finding.

### [LOW] SyncOptions.BackoffSleep doc lists only some of its uses

**Location:** internal/fabricengine/fabric.go:82

**Issue:** The field doc says it sleeps one backoff delay "before a missing branch or unreachable origin is recorded".
`settleRejection` already used it otherwise, and the diff adds the records side's re-push in `pushRecordsSide` (`pushanchored.go:213-234`), so the doc reads narrower than the field's contract.

## Rejected

- Correctness lens finding 2, a persistently failing tracking-ref write misreported as commits the push did not carry: the sentinel's own doc names a tracking ref the push left unmoved as one cause, a transient write failure clears on the next push's `recordLandedPush`, and a write that fails every time is a broken local repository whose error already points at the records branch.
- The assistant reviewer's claim that `PATTERN-pair-remote` requires a failure record and notice on these paths: that entry governs the order of the records and code sides and the teardown lock, which the retry keeps; the owed-push gap stands on the board entry's Expected alone and is carried in F1.

## Holistic notes

The retry is bounded: with the default three delays it pushes at most four times, and an empty delays slice returns on the first attempt before indexing.
The deferral check still precedes the origin lookup, so a fresh deferral reads as its `*PushFailedError`, and a gone branch still reaches the ended message only after `checkOrigin` judged its entry ended, as the ended subtests in `pushclasses_integration_test.go:702-744` pin.
`branchGoneOnOrigin` asks `ls-remote --heads origin refs/heads/<current branch>` in the records worktree, the right remote and ref for `main-weft`.
Sleeping inside the weft-side absorbing push lock matches `settleRejection` and `awaitOrigin`, which already sleep the same backoff under it.
The post-receive hook commits synchronously before the client's push returns, the old code returns the ended error on that path, and the new subtest does not duplicate `PrimePushesRecordsSideOnly`, which has no commit during the push.
The commit messages and `_lym/landing/summary.md` match the diff; no `Pattern-Departure:` trailer is present and none is needed.
