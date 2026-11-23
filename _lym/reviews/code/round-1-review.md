---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/battencli/wire.go#Teardown.Level
    summary: "With FABRIC_SKIP_GIT or FABRIC_SKIP_PUSH set, PushPairAnchored pushes nothing and returns nil, TipsOnOrigin then reads false, and the Worktree-Teardown loop re-pushes and re-takes the prime lock forever without a sleep or a Remove."
    fix: "In the Teardown.Push and Teardown.Level closures of internal/battencli/wire.go, read fabricengine.EnvSyncOptions() and, when SkipGit or SkipPush is set, make Push return nil without calling PushRun and Level return true, so the removal runs as it did before this change. Add a TestWorktreeTeardown or wire-level row that sets the skip option and asserts the row finishes Done. Also bound the Call loop in internal/battenshed/teardown.go so a Level that stays false after a push that returned nil yields Stuck naming the unreadable level, instead of spinning."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/wire.go#Teardown.Push
    summary: "No test pins the Teardown.Push and Teardown.Level wiring for a present pair: the mapping of ErrEntryEnded to ErrPushEntryEnded and of ErrHandedOver to ErrPushHandedOver, nor the real PushRun and TipsOnOrigin calls."
    fix: "Add an integration step beside stepTeardown_ResumedRowWhosePairIsGoneFinishesDone in internal/battencli/lifecycle_integration_test.go that runs the Worktree-Teardown row over a present pair whose records branch origin ended and asserts Done with the pair removed, and one over a taken-over run asserting the Stuck reason names the hand-over. Alternatively extract the error mapping into a function and cover it with a table test."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/dobbyreads.go#DobbyRuns
    summary: "Nothing asserts that the dobby tick or a resume attempts a pusher start: restartPusher in DobbyRuns, the prime RestartPusher in the beamcli Tick, and the startPusher closures in battencli and loomcli would pass every test if their calls were deleted, since spawnPusher fails under go test."
    fix: "Inject the start function so a test sees the call. For the dobby, give DobbyRuns and the hub Tick a seam for the start function and assert in internal/battencli and internal/beamcli tests that a tick calls it once for the prime and once for each owned pair that stands here. For the resume closures in internal/battencli/wire.go and internal/loomcli/wiring.go, assert through the same seam that the closure passes the prime target and, where the pair stands, the pair target."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/runpusher/start.go#tryLock
    summary: "The real tryLock, LockPath, NewDeps conditions, worktreeGone, pauseCovers, handedOver and RestartPusher have no test; TestRun and TestStart use fakes only, so a lock that does not exclude a second pusher passes every test."
    fix: "In internal/runpusher add a tier-1 test over a temp hub that takes tryLock(LockPath(target)) twice and expects the second acquired to be false, and that after release a third take succeeds. Add a worktreeGone row over an existing and a removed temp directory, and a RestartPusher row that asserts a failing start logs once per episode."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/prepush.go:1
    summary: "The file header and the guardedCommit comment still say the check runs before the run commits or pushes its work and that a status commit's caller reads its stop; status commits and commit-records now run no check."
    fix: "Reword the file header at the top of internal/loomcli/prepush.go and the guardedCommit doc comment to say the check guards the landing's commit and push and PR-Review's PushTask, and that the pusher runs it through guard before each push."
  - id: F6
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/slackengine/relay_test.go:1103
    summary: "The change from r.start() to r.settle() in the on-disk check row traces to no decision of the record; commit 2198f259c carries no trailer. It widens no assertion, so it is no loosened check."
    fix: "Move the relay_test.go change and its commit out of this branch into its own fix, or leave it and accept it as unrelated; no Loosened-Check trailer is needed since no assertion or limit is widened."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/pushclassified.go#checkOrigin
    summary: "The refPruned branch with an upstream set, the tracking ref absent and the branch still on origin pushes and skips holdBackRewound, but no PushPairAnchored row drives it."
    fix: "Add a row to TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide in internal/fabricengine/pushanchored_integration_test.go that prunes the code branch's tracking ref with pruneTrackingRef, leaves the branch on origin, commits locally, pushes, and asserts the push lands and origin holds the local tip."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/takeover.go#primePushTakeovers
    summary: "Two branches have no row: the runSlug appended when no unpushed trailer names it, and the merge of handedOver into an EndedSet that origin derived in retryWeftPush."
    fix: "Add rows to TestRejectedRecordsPush_ReadsTheTakeover in internal/fabricengine/rejectedtakeover_integration_test.go: a prime push naming a taken-over run none of whose commits are unpushed, asserting a TakeoversError for it; and a prime push where origin ended one run and another was handed over, asserting both runs' commits are set aside and the rest land in order."
  - id: F9
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/battencli/commitstatus_test.go#TestNewCommitStatusSeam
    summary: "The ResumedStartRowCommitsAndStartsThePusher row pins what StartRowsEachCommitAndStartThePusher and ChangedPairCommitsAgain already pin: every row commits and starts the pusher."
    fix: "Delete the ResumedStartRowCommitsAndStartsThePusher row from the tests table of TestNewCommitStatusSeam in internal/battencli/commitstatus_test.go."
---

### [MEDIUM] Worktree-Teardown spins forever when pushes are off

**Location:** `internal/battencli/wire.go#Teardown.Level`, loop in `internal/battenshed/teardown.go#Call`
**Issue:** `PushPairAnchored` returns at once with no push when `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` is set (`pushanchored.go:184`).
`Teardown.Push` then returns nil, and `TipsOnOrigin` reads false because the branch has no upstream or a stale ref.
`Call` loops: lock, level read false, release, push, with no sleep, and never reaches `Remove`.
Before this change a teardown with pushes off removed the pair.
D5 only says a run with pushes off starts no pusher; it does not license a teardown that never finishes.

### [MEDIUM] The teardown's push wiring is untested

**Location:** `internal/battencli/wire.go#Teardown.Push`
**Issue:** `TestWire_BuildsLazilyForANonexistentTaskWorktree` checks the two closures are non-nil, and the new integration step covers only a gone pair.
The sentinel mapping for an ended entry and a hand-over, and the real `PushRun` and `TipsOnOrigin` calls for a present pair, fail no test when broken.
A broken ended-entry mapping would halt the teardown as a refused push.

### [MEDIUM] Dobby tick and resume restarts are untested

**Location:** `internal/battencli/dobbyreads.go#DobbyRuns`
**Issue:** D5's acceptance that a tick starts a pusher for a prime or pair ahead, and that a resume starts it before the run acts, is pinned only at `starter.start` and at the ordering of the injected `startPusher` closure.
The closures that call `runpusher.Start` or `StartIfAhead` in `DobbyRuns`, the beamcli `Tick`, `battencli/wire.go` and `loomcli/wiring.go` run under tests but would pass if the calls were removed.

### [MEDIUM] The pusher lock and conditions are exercised only through fakes

**Location:** `internal/runpusher/start.go#tryLock`
**Issue:** D3's "at most one pusher per worktree" and "a pusher whose worktree was removed exits" are pinned against fake `Lock` and `WorktreeGone` seams.
The real lock path, `NewDeps` conditions, `pauseCovers`, `handedOver` and `RestartPusher` have no test, and the package has no coverage map.

### [LOW] Stale pre-push comments in loomcli

**Location:** `internal/loomcli/prepush.go:1`
**Issue:** The header and the `guardedCommit` comment describe a check on every status commit and push of the run.
After this change only the landing, PR-Review's `PushTask` and the pusher run it.

### [LOW] Unrelated slackengine test synchronisation

**Location:** `internal/slackengine/relay_test.go:1103`
**Issue:** The swap of `r.start()` for `r.settle()` traces to no decision, and commit `2198f259c` carries no trailer.
It adds a wait before the row's hint and widens no assertion, so no `Loosened-Check:` is owed.

### [LOW] Pruned-ref push with a live origin branch has no row

**Location:** `internal/fabricengine/pushclassified.go#checkOrigin`
**Issue:** The `refPruned` path pushes and skips the rewind check.
The new rows cover only the ended-entry drop and the local reads.

### [LOW] Two prime-takeover branches have no row

**Location:** `internal/fabricengine/takeover.go#primePushTakeovers`
**Issue:** The appended `runSlug` and the merge of handed-over slugs into an origin-derived `EndedSet` are driven by no row.

### [NIT] Redundant seam row

**Location:** `internal/battencli/commitstatus_test.go#TestNewCommitStatusSeam`
**Issue:** `ResumedStartRowCommitsAndStartsThePusher` repeats the coverage of `StartRowsEachCommitAndStartThePusher` and `ChangedPairCommitsAgain`.

## Rejected

- Correctness fork F1 (`HeadContains` errors on an absent code tip in `runread.go`): `gitrepo.HeadContains` returns `false, nil` on `plumbing.ErrObjectNotFound` (`internal/gitrepo/blobread.go:82-84`), so an absent tip reads as not held and the old `CodeTipMissing` path stands.
- Affected-set fork F2 (removed assertions without `Loosened-Check:`): the removed rows belong to the retired push, retry and start-row behaviours, and the relay change adds a wait without widening an assertion.
- Scope fork F2 and F3 (tracking-ref deletion of an ended branch, `ErrEntryEnded` skip): both are forced by D5's "a pruned tracking ref reads level" and by the teardown's need to finish, and the teardown's loop depends on them; no unrequested behaviour.
- Tests fork finding on `DeleteRemoteTrackingRef` having no direct test: the `OwedUnrecorded_A...EndedEntry...` rows reach it through `requireTrackingRefPrunedWithUpstreamSet`.
- Affected-set fork F3 (`PATTERN-pair-remote.md` silent on the records-before-code window): the fork raised nothing, and no declared text requires it.
- Declared PATTERN changes: all three entries land with their declared text in `PATTERN.md` and their backgrounds, the glossary line lands, and no `pattern/` text still states a transition push before the run acts; nothing to raise.
- Focus on `runproc.EndSubject` sparing `RolePush`: the sparing already exists at the base (`internal/runproc/end.go:100`), and the pusher registers under `runproc.HubOwner` with `RolePush` in `start.go`.
