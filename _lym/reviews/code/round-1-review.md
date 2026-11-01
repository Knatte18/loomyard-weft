---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: PATTERN.md:30
    summary: The new entry says every push to origin goes through gitrepo under the lock, but the archive tag push in archive.go is a raw gitexec.Run push outside it, with no Pattern-Departure trailer.
    fix: In PATTERN.md line 30, narrow the entry to calls that write remote-tracking refs, saying a tag push and a fetch into a private probe clone are outside it, next to the existing --refmap= exemption. Or route the tag push in internal/fabricengine/archive.go (tagAndPushArchiveTip, the pushTag closure) through a gitrepo push method that takes the lock. Pick one so the entry and the code agree.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gitrepo/remoterefslock_integration_test.go:68
    summary: No test fails deterministically if Fetch, Pull, FetchWithArgs or a push method stops taking the lock; the only test that exercises it is a probabilistic concurrency test that also passes without the lock.
    fix: In remoterefslock_integration_test.go add one table-driven test over Fetch, FetchWithArgs, Pull, PushRebaseFree, DeleteRemoteBranch, DeleteRemoteBranchLeased and UpdateRemoteBranchLeased. For each row take the lock with HoldRemoteRefs, start the call in a goroutine, assert it has not returned, release, then assert it completes. Use a signal rather than a real-time sleep for the not-returned check.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/originreconcile_integration_test.go:227
    summary: Nothing pins that the locks are held through the branch verdicts, which the record asks for so a push in flight is never judged a rewind; releasing right after the fetch would pass.
    fix: In the same subtest, or a new row of it, make the verdict observable while the locks are held. For example, wrap judgeBranch's first tracking-ref read in a test seam like the existing call hook, or hold a lock with HoldRemoteRefs and assert that ReconcileOrigin blocks until released and then judges the branch against the pushed head. The case must fail if judgeAgainstOrigin released the locks after fetchMovedHeads.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gitrepo/push.go:153
    summary: PushRebaseFree is still documented as Lock-free although each attempt now runs under the remote-refs lock.
    fix: In the PushRebaseFree doc comment in internal/gitrepo/push.go replace the final line "Lock-free." with "It takes no push-serialising lock of its own; each attempt runs under the repository's remote-refs lock.", or delete the line.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/archive.go:160
    summary: resolveArchiveTip reads FETCH_HEAD after FetchWithArgs has released the lock, so another locked fetch in the same worktree can rewrite FETCH_HEAD first and the archive tag could be made at the wrong tip.
    fix: In resolveArchiveTip, resolve the fetched tip without FETCH_HEAD, for example by fetching into an explicit ref the caller owns, or have gitrepo expose one locked call that fetches and returns the resolved FETCH_HEAD commit while holding the lock. Keep the error text of the fetch unchanged.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/originreconcile.go:747
    summary: Both repositories' locks are held across lsRemotePair's whole retry loop including its sleeps, so an unreachable origin blocks every fetch and push of both repositories for the retry bound.
    fix: Accept and say so in judgeAgainstOrigin's doc comment that the locks are held through the ls-remote retries, bounded by ReconcileOptions' attempts and deadline. If a shorter hold is wanted, take the locks only around the final successful ls-remote attempt, which the record's wording allows only if the read and the verdict still share one hold.
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/add.go:415
    summary: No test fails if a fabricengine fetch site goes back to an unlocked gitexec.Run fetch, though the record asks that every such fetch be under the lock.
    fix: Add to cmd/lym/gitrepoboundary_test.go, or beside it, an AST scan of internal/fabricengine non-test files that fails on a gitexec.Run or RunGit call whose first argument list starts with "fetch" and has no "--refmap=" element. The two --refmap= sites in remoteleftover.go stay allowed.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/originreconcile_integration_test.go:227
    summary: Lock release on the unreachable path, the error path and the judge-only scope has no test.
    fix: Add rows to the same subtest using a hook that returns a transport failure, and a ReconcileScope with JudgeOnly true, each asserting that all lock paths are free after ReconcileOrigin returns.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: PATTERN.md:30
    summary: The new entry carries no (test) marker and no enforcing test, so a raw fetch outside gitrepo can be reintroduced unnoticed.
    fix: Land the scan from F7 and mark the entry (test) in PATTERN.md line 30; add a short pattern/PATTERN-remote-refs-lock.md background and link it only if the entry grows past one line. If F7 is not taken, leave the entry as is.
---

## Findings

### [MEDIUM] Entry wording and the archive tag push disagree

**Location:** PATTERN.md:30; internal/fabricengine/archive.go:128
**Issue:** The entry binds "pushing to origin" to gitrepo's methods under the lock and exempts only a `--refmap=` fetch. `tagAndPushArchiveTip` still runs `gitexec.Run` with `push`, outside the lock and outside gitrepo. A tag push writes no tracking ref, so the code is probably right and the entry is too broad. The record asks for fetches into tracking refs only.

### [MEDIUM] The lock itself is not pinned by a deterministic test

**Location:** internal/gitrepo/remoterefslock_integration_test.go:68
**Issue:** `TestRemoteRefsLock_ConcurrentFetchesAndPushesOfOneRepositoryAllSucceed` asserts success of concurrent calls. `PushRebaseFree` already retries "cannot lock ref", so the test can pass with the lock removed, and no test covers `Pull`, `FetchWithArgs`, `Push`, `PushCoalesced` or the three remote-branch methods.

### [MEDIUM] The hold through the verdicts is unpinned

**Location:** internal/fabricengine/originreconcile_integration_test.go:227
**Issue:** The new subtest observes the locks only at `ls-remote` and `fetch` calls and after return. A `judgeAgainstOrigin` that released the locks before `judgeBranch` passes. The record's requirement that a push in flight finish before origin is judged rewound rests on that span.

### [LOW] PushRebaseFree doc says Lock-free

**Location:** internal/gitrepo/push.go:153
**Issue:** The method takes the remote-refs lock on every attempt, which the file header at line 2 says.

### [LOW] FETCH_HEAD is read after the lock is released

**Location:** internal/fabricengine/archive.go:160
**Issue:** `resolveArchiveTip` fetches under the lock, then runs `rev-parse FETCH_HEAD^{commit}` outside it. A second locked fetch in the same worktree can overwrite FETCH_HEAD in between. The race predates the diff, but the diff touched this call and left the window.

### [LOW] Locks are held through the ls-remote retry sleeps

**Location:** internal/fabricengine/originreconcile.go:747
**Issue:** An unreachable origin holds both locks for the whole retry bound, blocking fetches and pushes of both repositories. The effect is bounded and matches the record's "read and verdict under the same lock", but the doc comment does not say it.

### [LOW] No test pins the fabricengine fetch sites

**Location:** internal/fabricengine/add.go:415
**Issue:** Eight fabricengine sites were converted to `FetchWithArgs`; no test fails if one reverts.

### [LOW] Release paths of the reconcile are unpinned

**Location:** internal/fabricengine/originreconcile_integration_test.go:227
**Issue:** The unreachable, error and judge-only returns are not exercised against the locks. The `defer` makes them likely correct.

### [LOW] New entry has no enforcing test

**Location:** PATTERN.md:30
**Issue:** Other entries about git boundaries carry a `(test)` marker and a scan. This one relies on review alone. `PATTERN-parent-lock` and others also lack both, so this is a gap in enforcement, not a rule violation.

## Rejected

- Lock re-entrance deadlock inside `judgeAgainstOrigin` (focus 1): `fetchMovedHeads` and `lsRemoteHeads` call `gitexec` directly and `judgeBranch` makes only `gitrepo` reads, so nothing reaches a method that takes the lock. Not a defect.
- Warp and records repositories sharing one common git dir (focus 1; code-correctness NIT): the code store is the warp repo and the records store is the weft repo, resolved separately in `originTargets` by `RecordsRepoRoot`. Distinct repositories, so one lock file is not taken twice.
- Lock order and deferred-release order (focus 2): the order is fixed warp then records, nothing takes them reversed, and the defers run last-in first-out at return after the final verdict. Not a defect.
- Splitting a push and its `pull --rebase` across separate lock holds (focus 4): each git call completes under a hold, so no push is in flight when the lock is free. The window between holds only lets another fetch land, which the rebase adapts to. The record's requirement is met.
- `Pull` and `Fetch` exit-code handling (focus 5): on a lock failure `err` is returned before `code` is read, and on a non-zero exit `code` is set and handled as before. Error text is unchanged.
- Stale lock file from a killed process (focus 6): the lock is an `flock`, which the kernel drops with the process, so no stale holder survives. The wait is unbounded by design, as the record asks for waiting rather than failing; a live holder is bounded by git's own run.
- `res.Branches` is now nil when `judgeBranch` errors (code-correctness finding 3): `ReconcileOrigin` documents that on this error nothing is judged or moved, so partial verdicts were never a contract.
- Stale "10 non-test .go files" comment in `gitrepoboundary_test.go` (affected-set note): the floor is 5 and the count is only a comment; the guard still holds.
- Code-tests NIT on shared `publishedClone` setup: shared setup, not duplicate coverage.
- `target.tracking` is read in `originTargets` before the lock is taken: it only chooses which heads to fetch, and a stale value at worst fetches a head that already matches. `judgeBranch` reads the live tracking ref under the lock.
