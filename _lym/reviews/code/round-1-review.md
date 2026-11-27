---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/runpusher/parent.go:279
    summary: "A landing settled on origin whose run reads owned elsewhere leaves the parent's refused-push record standing, so the parent target stays refused for good"
    fix: "In settleOnOrigin's `owner != \"\"` branch, after removeLanding, call clearUnlessDropped(deps) and write no new record; a later pass records again any cause that still stands. Add a TestParentPush row: the run is owned elsewhere, a pass writes held_back, the tip is pushed by hand, SettleRefused leaves no failed-push record and a later landing on the same parent is pushed."
    origin: lens:code-scope
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/runpusher/parent.go:313
    summary: "The parent step removes a landing from the ledger before the reset or hand-over it owes, so a failed one is never retried and nothing is recorded"
    fix: "Make the owed reset and hand-over survive a failure, in dropLandedElsewhere and in settleOnOrigin's owned-elsewhere branch: keep in the parent entry what is still owed (or keep the landing until the step succeeded, noting that pairteardown.HandOver keeps the pair while a landing stands) and retry it on every pass and in SettleRefused. A refused `git reset --keep` must write a held_back record naming the commit and the way forward. In pushParent, build the Dropped blockers before the `len(landings) == 0 && len(blockers) == 0` early return, for each dropped commit the parent's head still carries. Add rows for a reset refused over a dirty tracked path and for a failing hand-over."
    origin: lens:code-correctness
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/beamcli/dobbydeps.go:264
    summary: "The dobby tick never starts a beam parent's pusher, so its landings are never pushed or settled after a refusal is cleared, a reboot or a pause"
    fix: "In restartHubPushers, for each target of parentTargets that settleIfRefused reports as not refused, call runpusher.RestartPusher with start(target), keyed by runpusher.LockPath(target). Give TestRestartHubPushers a row in which the unrefused beam target is handed to start."
    origin: lens:code-correctness
  - id: F4
    severity: BLOCKING
    class: design
    location: internal/fabricengine/fabric.go:192
    summary: "Fabric.PushBranch and Fabric.LocalOnlyCommits lost their last production callers and are now called only by tests, against PATTERN-test-only-code"
    fix: "Delete Fabric.PushBranch (internal/fabricengine/fabric.go) and Fabric.LocalOnlyCommits (internal/fabricengine/parenttips.go), remove or retarget the tests that call them, and drop Fabric.PushBranch from the list in internal/fabricengine/doc.go."
    origin: lens:code-affected-set
  - id: F5
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/pushclassified.go:534
    summary: "A parent push refused by branch protection records only a retry as its way forward; card 12 requires the record to carry the protected-branch way forward"
    fix: "When a parent push (PushPrimeCodeTip, and a beam parent's push) fails with ProtectedBranchRefusal, set the record's WayForward to the advice the removed protectedParentWayForward gave: set beam.pull_request to trail, named in full through the config key-name helper, or land the work through a pull request by hand, then `git fetch origin` and `git reset --keep origin/<parent>` in the parent's worktree. Make FailedPushWayForward return that field for such a record, since its default branch ignores it, and add a TestPushPrimeCodeTip row with a declining hook."
    origin: handler
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/runpusher/parent.go:435
    summary: "A landing whose commit a rebase removed from the parent keeps its old SHA, and pushParent still pushes it to origin and marks the run done"
    fix: "In refreshTips or pushParent, when LandingCommits lists no commit for a landing's run, the parent's head does not carry landing.Tip and origin does not hold it, do not push that tip: take the landing out of the push and either drop it with its records held or record a held_back refusal naming the removed commit and the way forward. Add a TestParentPush row that resets the landing commit away before the pass."
    origin: lens:code-correctness
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/loomcli/landingdeps.go:946
    summary: "A Finalize resume replaces the run's landing with a newer records commit, releasing the earlier held commit to origin before the parent holds the work"
    fix: "In RecordLanding (or recordHeldLanding), when a landing of the run already stands and its RecordsCommit is still unpushed and an ancestor of the new one, keep the earlier RecordsCommit. Add a row that records twice and asserts a pair push still stops short of the first commit."
    origin: lens:code-correctness
  - id: F8
    severity: MEDIUM
    class: design
    location: internal/tenderengine/resume.go:288
    summary: "A hub lift pushes no pair a pause held back, since pairsToPush reads a pending flag only the no longer spawned `lym fabric push` child wrote"
    fix: "Change pairsToPush and the PushDeferred wiring in internal/tendercli/deps.go to decide from what the pusher leaves, for example fabricengine.RunCommitsAhead of the pair or its push-ledger entry, and start the pusher for each such pair. Update the lift's help text and the spawnOwed doc, and cover a pair with no batten run."
    origin: lens:code-affected-set
  - id: F9
    severity: MEDIUM
    class: design
    location: internal/webstercli/reset.go:186
    summary: "D1 keeps webster reset's records push synchronous with an owed record, but it now goes to the quiet pusher, which writes none"
    fix: "After fabricSync in the reset verb, push the pair's records synchronously through pairteardown.PushRun (OwedUnrecorded unset), so an owed failure is recorded and holds the run as D1 says. Then state that push in pattern/PATTERN-pair-remote.md's synchronous list, in place of `webster reset`'s move of the code branch alone, and restore the owed-record hold in pattern/PATTERN-run-fsm.md."
    origin: lens:code-scope
  - id: F10
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pull.go:648
    summary: "A prime that holds an owed tip while origin has new commits pulls successfully without moving, so Land deploys code lacking origin's commits"
    fix: "In primeAheadByOwedTips, report true only when the tracking ref is an ancestor of the local head (ahead, not diverged), so a diverged prime is judged by the existing diverged path. Add a TestLandDeps_PullWithAnOwedLandingPush row with a second clone's commit on origin."
    origin: lens:code-correctness
  - id: F11
    severity: MEDIUM
    class: scope
    location: internal/loomcli/parentpush_integration_test.go:699
    summary: "TestParentPush lacks card 11's rows for the owner pull, for settle and drop of a run owned elsewhere, and for partial and whole hold-back"
    fix: "Add the rows card 11 names: an owner entry on origin's board only, for a pass and for SettleRefused; a landing on origin and a dropped landing whose run is owned elsewhere (no done write, HeldRecords, hand-over, no refused record, origin never reaches the held commit); an older tip still pushed while a newer run is held back; a beam parent held back whole; a beam parent's only landing dropped then rebased; and a pair removed after a drop with a newer landing."
    origin: lens:code-tests
  - id: F12
    severity: MEDIUM
    class: scope
    location: internal/landingshed/finalize_test.go:190
    summary: "Acceptance behaviours have no failing test: no landing entry with pushes off, the sync-status verb's output, the summary's wired refusal read, and a pair's network wait in DobbyRuns"
    fix: "Give the pushes-off Finalize row a recording LandLocally and assert it is not called. Add a test of `lym fabric sync-status` (the line, `sync: ?` with exit zero, the base interval from config). Drive hubPushSummary's Refused over a real prime code-branch record and its clearing. Add DobbyRuns rows for a pair's wait record and for a failed network status under a held driver lock with no record yet."
    origin: lens:code-tests
  - id: F13
    severity: LOW
    class: consistency
    location: internal/fabricengine/pushclassified.go:370
    summary: "Behaviours with no row: a rejected capped push, ResetParentToOrigin's refusals, and the drivers' and pusher's configured timing wiring"
    fix: "Add a push row with a records hold or code ceiling against a moved origin asserting no replay or reset; rows for ResetParentToOrigin over a dirty touched path and a branch with no upstream; and assertions that goDriverDepsFor, batten's driverDepsFor and pusherDeps carry a configured ceiling, interval and wait record path."
    origin: lens:code-tests
  - id: F14
    severity: LOW
    class: design
    location: internal/fabricengine/pushclassified.go:534
    summary: "A diverged beam-parent push does not name the landing commits origin lacks, as D5 asks of a refused parent push"
    fix: "Pass the beam parent's landing commits into the push options on the parent pusher's beam push, so record() names them and its way forward uses `lym fabric push`; record() currently passes prime=true to codeDivergedWayForward, so make that follow the job's prime flag."
    origin: lens:code-scope
  - id: F15
    severity: LOW
    class: design
    location: internal/battenshed/deps.go:361
    summary: "ErrPushEntryEnded and ErrPushHandedOver have no producer left, so the teardown's branches over them are unreachable"
    fix: "Delete both sentinels, the entryEnded path and the switch in worktreeTeardownProducer.pushPair, the tests that inject them, and their mentions in the TeardownDeps.Push and NewWorktreeTeardown docs."
    origin: lens:code-affected-set
  - id: F16
    severity: LOW
    class: design
    location: internal/fabricengine/pushclassified.go:476
    summary: "unmadeOriginCommit passes every reflog entry as a command-line argument, which exceeds the Windows command-line limit on a long-lived branch"
    fix: "Feed the revisions to `git rev-list --stdin` through gitexec.RunStdin, with the excluded ones written as `^<sha>` beside `^<named>`, keeping `--branches` as an exclusion."
    origin: handler
  - id: F17
    severity: LOW
    class: consistency
    location: internal/loomcli/doc.go:233
    summary: "Docs and help text still describe the old push behaviour, and one new line says the landing target has no push yet"
    fix: "Correct internal/loomcli/doc.go:233; the one-lock-per-records-worktree lines in internal/lymdirs/dirs.go and doc.go; `one per records worktree` in pattern/PATTERN-run-fsm.md:27; the runpusher line in docs/overview.md; the detached `lym board sync` mentions in internal/boardengine/board.go, internal/hubreconcile/doc.go and internal/tendercli/doc.go; the help of `fabric sync` and `fabric add`; and the detached-child comment in internal/fabriccli/roleguard.go."
    origin: lens:code-affected-set
  - id: F18
    severity: NIT
    class: consistency
    location: pattern/PATTERN-lym-files.md
    summary: "The list of run-related caches under .lym omits the push ledger, the drive-wait record and the merge-in base file"
    fix: "Add the three to the cache list under `Durable and ephemeral state`."
    origin: lens:code-affected-set
---

### [BLOCKING] Refused record outlives a landing settled for a run owned elsewhere

**Location:** `internal/runpusher/parent.go:279`
**Issue:** The `owner != ""` branch of `settleOnOrigin` removes the landing and hands the run over, and never clears the record.
`clearRecordedGap` clears only unrecorded-commit reasons, so a `held_back` record written for that owner stays.
The parent target then starts no pusher, later landings on it are never pushed, and a GitHub landing halts.
D5 says a landing settled "however it got there" clears the record; the arbiter's card 11 ruling forbids only writing a new one.

### [BLOCKING] A failed reset or hand-over is lost

**Location:** `internal/runpusher/parent.go:313`
**Issue:** `dropLandedElsewhere` moves the run to `Dropped` and only then calls `ResetToOrigin` and `HandOver`.
If the reset is refused, for example over a dirty tracked path in the parent's worktree, no later pass has a landing to retry it from.
`pushParent` returns before it builds the `Dropped` blockers when no landing is left, so no record or notice is written.
The duplicate landing commit stays on the local parent, and the next Finalize on it is Stuck as ahead.

### [BLOCKING] Beam parent pushers are never restarted

**Location:** `internal/beamcli/dobbydeps.go:264`
**Issue:** The pair targets of `parentTargets` are only passed to `settleIfRefused`.
The other tick-time start, in `DobbyRuns`, covers batten runs' pairs, and a beam pair is none.
After the named `lym fabric push` clears a beam's refusal, nothing settles the member's landing: no `done` write, its records stay held and its teardown refuses.
D3 requires the next tick to drop the entry or start the pusher.

### [BLOCKING] Production methods only tests call

**Location:** `internal/fabricengine/fabric.go:192`, `internal/fabricengine/parenttips.go:73`
**Issue:** The diff removed the last production callers of `Fabric.PushBranch` and `Fabric.LocalOnlyCommits`.
A grep over `internal` and `cmd` outside tests finds only the definitions and a doc line.
The dead-code guard runs only under `upkeep`, so no task gate reports it.

### [MEDIUM] Protected-branch way forward is gone

**Location:** `internal/fabricengine/pushclassified.go:534`
**Issue:** Card 12 removes `protectedParentWayForward` on the condition that the pusher's refused-push record carries that way forward.
Nothing builds it.
A protected rejection records as `other`, and `FailedPushWayForward`'s default tells the reader to run `git push` again, which the protection refuses every time.

### [MEDIUM] A commit removed by a rebase is still pushed

**Location:** `internal/runpusher/parent.go:435`
**Issue:** `refreshTips` updates a tip only while `LandingCommits` still lists the run, and `pushParent` never checks that the parent's head carries it.
`PushPrimeCodeTip` pushes by SHA, so an orphaned commit that fast-forwards origin lands there.

### [MEDIUM] Resume releases the first held records commit

**Location:** `internal/loomcli/landingdeps.go:946`
**Issue:** `RecordLanding` deletes the run's earlier landing and stores the new records commit.
The hold then starts at the newer commit, so the earlier one may be pushed while origin's parent lacks the work, against D5.

### [MEDIUM] A lift pushes nothing the pause held

**Location:** `internal/tenderengine/resume.go:288`
**Issue:** `pairsToPush` skips each pair for which `PushDeferred` is false.
That flag was written by the detached `lym fabric push` child, which no spawn starts now; the pusher exits on a pause and leaves no flag.
A pair with no batten run stays unpushed until its next commit.

### [MEDIUM] `webster reset`'s records push writes no owed record

**Location:** `internal/webstercli/reset.go:186`
**Issue:** D1 lists this push among those that keep their own behaviour, "whose owed record holds the run at its transition".
It is `Fabric.Commit`'s spawn, which now starts a pusher that pushes with `OwedUnrecorded`.
The run still waits on the missing code tip, but without the record D1 names.
Plan card 7 routes every spawn to the pusher, and no trailer or added entry settles the difference.

### [MEDIUM] A diverged prime pulls as level

**Location:** `internal/fabricengine/pull.go:648`
**Issue:** `primeAheadByOwedTips` checks only that each local commit past origin is an owed tip, not that origin holds nothing new.
`pullWarpArm` then returns nil with the prime unmoved, and Land deploys it.
D5 lets such a pull read level only for a branch that is ahead.

### [MEDIUM] Card 11's parent-pusher rows are missing

**Location:** `internal/loomcli/parentpush_integration_test.go:699`
**Issue:** No row puts an owner on origin's board only, settles or drops a run owned elsewhere, or shows a partial or whole hold-back.
The added entries require the first two, and the defect in the first finding sits on one of these untested paths.

### [MEDIUM] Acceptance behaviours with no failing test

**Location:** `internal/landingshed/finalize_test.go:190`
**Issue:** The pushes-off row leaves `LandLocally` nil, so it passes whether or not `PushSkipped` guards the call.
The `sync-status` verb, `refusedPushWayForward` with `recordNamesTarget`, and the pair branches of `readRunWait` have no test.

### [LOW] Further coverage gaps

**Location:** `internal/fabricengine/pushclassified.go:370`
**Issue:** No row rejects a capped push, exercises a refusal of `ResetParentToOrigin`, or checks that the configured timing reaches the drivers and the pusher.

### [LOW] Beam divergence names no landing commit

**Location:** `internal/fabricengine/pushclassified.go:534`
**Issue:** Only `PushPrimeCodeTip` sets `opts.landings`, so a diverged beam-parent push gets the generic merge line.

### [LOW] Unreachable teardown sentinels

**Location:** `internal/battenshed/deps.go:361`
**Issue:** `teardownPushError` was the only producer of both sentinels, and the new `Push` returns neither.

### [LOW] Reflog passed as arguments

**Location:** `internal/fabricengine/pushclassified.go:476`
**Issue:** Each distinct reflog entry adds an argument of 40 characters.
A few hundred entries within the reflog expiry pass the Windows limit, and the rejected push then fails with an unnamed error.

### [LOW] Stale descriptions

**Location:** `internal/loomcli/doc.go:233`
**Issue:** The listed docs, comments and help texts describe the per-records-worktree pusher, the detached sync child or a landing target without a push.

### [NIT] Cache list omits the new files

**Location:** `pattern/PATTERN-lym-files.md`
**Issue:** The background enumerates the run-related caches under `.lym` and leaves out the three this change adds.

## Rejected

- The code-tests lens rated the missing owner-pull and owned-elsewhere rows BLOCKING: kept as one MEDIUM coverage finding (F11), since the code on the owner-pull path is present and the defect on the owned-elsewhere path is raised on its own as F1.
- The code-affected-set lens's aside that the prime and landing pushers read no pause: not kept as a finding, since it was not traced and the cards give those targets no pause read.
