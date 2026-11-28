---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/07-push-side-core.md:21
    summary: "TestPushClassified_Snapshot (card 7) and TestReconcileOrigin_Snapshot (card 9) are planned in package fabricengine on a hubforge pair, but internal/hubforge imports internal/fabricengine, so either test file is an import cycle and fails its card's build gate"
    fix: "In card 7, replace the Create line for `plan:internal/fabricengine#TestPushClassified_Snapshot` with `plan:internal/fabricengine_test#TestPushClassified_Snapshot` -> `func TestPushClassified_Snapshot(t *testing.T)`, add the Create line `plan:internal/fabricengine#PushClassifiedSnapshotForTest` -> `func PushClassifiedSnapshotForTest(hubPath, side, path string, opts SyncOptions) (fabriccore.Snapshot, error)`, and state that this export_test.go accessor runs pushClassified's runRule and returns its snapshot. Rewrite the Tests bullet at line 153 to say the test is in package fabricengine_test and reads the snapshot through that accessor. In card 9, make the same change at line 19: replace the line with `plan:internal/fabricengine_test#TestReconcileOrigin_Snapshot`, add `plan:internal/fabricengine#ReconcileJudgeSnapshotForTest` -> `func ReconcileJudgeSnapshotForTest(l *lymcwd.Location, scope fabriclayout.ReconcileScope, opts ReconcileOptions) (fabriccore.Snapshot, error)`, and rewrite the matching Tests bullet. lym loom validate-plan --overlay accepts both changes."
  - id: F2
    severity: BLOCKING
    class: decision
    origin: lens:guard-reach
    location: _lym/plan/07-push-side-core.md:108
    summary: "Card 7 maps a protected-branch refusal to OutcomeProtected ahead of rejected and records it as other at once, while today it is a rejection: it runs one retry (repushOverOriginContained's fetch and push), then failRejection's judgment fetch, which writes 'a fetch found origin holding nothing this branch lacks', or not_fast_forward when origin is ahead. The record text, the class and the git process count all change, against D5"
    fix: "In card 7, rewrite the outcomeClassOf bullet (line 84) and the verdict bullet (line 108) so that a protected refusal keeps today's path. The side's first push that is a protected refusal is a rejection that runs the SeamRejectionRetry call. A retry outcome that is a protected refusal stops the retry loop at once, as rejectionRetryKind's not-retryable answer does. The verdict of that unsettled rejection reads the judgment facts as any other unsettled rejection does: failRejection's fetch, its class and its error text. protectedParentWayForward stays on an other record of a push that carried landings. Add two TestPushSide rows: a protected first push runs one retry call and then the judgment read; and a protected rejection with origin holding commits the branch lacks reads not_fast_forward."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/07-push-side-core.md:93
    summary: "Card 7 maps gitrepo.ErrPushRetriesExhausted to OutcomeTransportFailure, and PushSide pushes every transport failure again after each backoff delay. Today retryTransportPush re-pushes only what gitexec.IsTransportFailure classes as transport, and an exhausted push wraps a refusal's GitError, not a transport one. The loop gains rounds and git processes, against D3's 'no loop runs more rounds than today's code retries' and D2"
    fix: "In card 7, add the Create line `plan:internal/fabriccore#OutcomeRetriesExhausted` -> `const OutcomeRetriesExhausted OutcomeClass = \"retries_exhausted\"` and have outcomeClassOf map gitrepo.ErrPushRetriesExhausted to it, ahead of the transport case. State that the transport loop at line 93 never pushes it again, and that JudgePush records it as `transport`, as classifyFailure does today. Add a TestPushSide row (an exhausted push is pushed once and records transport) and a TestOutcomeClassOf row for the new class. lym loom validate-plan --overlay accepts the added line."
  - id: F4
    severity: MEDIUM
    class: decision
    origin: lens:guard-reach
    location: _lym/plan/04-origin-reads-seams.md:40
    summary: "Card 4 routes originStatusAtTip through originStatusBytesAtTip, which refuses an unset SeamRunRecordPaths. ReadUpstreamTakeover reaches it inside retryWeftPush's takeoverStop after the first push ran, so an unset seam surfaces mid-flow, joined to the rejection and written as an other failed-push record with a mail, not refused before acting as D13 requires. The same unguarded read sits in OriginParentRunStatus and OriginBattenStatus (handover.go:200,215), yet card 4 calls refuseUnsuppliedSeams the refusal every verb of this part runs first"
    fix: "Choose one. (a) Cards 7 and 8 run refuseUnsuppliedSeams(fabriccore.SeamRunRecordPaths) before the push lock in PushPairAnchored and coalescePushSides, the push verbs whose records retry reads the takeover. Card 4's sentence at line 35 then names exactly the verbs that run it, and the handover verbs' own pre-action refusal is left to part 4. (b) Have originStatusAtTip keep reading the status record's tree path itself, so the part-4 paths gain no refusal. That adds a shedrun import to handover.go (checked: it imports shedengine and not shedrun today), which extends debt D13 cuts. Record the choice in card 4's Intent."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/03-required-seam-installer.md:55
    summary: "Card 3's search for tests that would meet the refusal greps only ReconcileOrigin, OriginRunStatus and OriginCodeTip in internal/fabricengine. It misses the readers card 4's move makes refuse: OriginParentRunStatus and OriginBattenStatus (handover.go:200,215), and ReadUpstreamTakeover (takeover.go:95) reached through every records push retry. It also misses tests in the other packages that call ReconcileOrigin. Line 45 also says only cards 4, 9 and 10 read the seams, while card 11 moves tryPrimeLock and recordedPairs onto them"
    fix: "In card 3, replace the grep at line 55 with one that covers the indirect readers and every package: `grep -rlE 'ReconcileOrigin|OriginRunStatus|OriginCodeTip|OriginParentRunStatus|OriginBattenStatus|ReadUpstreamTakeover|PushPairAnchored|CoalescePushRequest|CoalescePushBothAt|DetachedPush' --include=*_test.go internal cmd`. Keep the rule that a hit whose hub no internal/hubforge builder makes calls hubforge.InstallRequiredSeams first, and name in the Intent the files the search found. Change line 45 to 'cards 4, 9, 10 and 11'. Unverified: whether any such hit exists, since the reviewer's search covered only direct calls in test files that build hubs with fabricengine.CloneHub or fabriccli.CloneAndWire."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/12-board-test-moves.md:12
    summary: "The rows card 12 moves into TestPush_BoardReplay call fabricengine_test fixtures that boardengine_test cannot see. Those are newFailureFixture and newFailureFixtureAt, recordsFixture with its boardBranch and originCommit methods, fakePushNotifier with its options, requireOneFailedPush, requireNoFailedPush, bareBranchTip, boardEntry, withBoardTitle, boardFileOf and boardTitles. The card creates only reconcileHub, newReconcileHub, writeRunStatus, teardownRecorder and requireEntry, so the moved tests do not compile as written"
    fix: "In card 12, add these Create lines after line 12: `plan:internal/boardengine_test#failureFixture` -> `type failureFixture struct`; `plan:internal/boardengine_test#newFailureFixture` -> `func newFailureFixture(t *testing.T) (*failureFixture, *fakePushNotifier)`; `plan:internal/boardengine_test#fakePushNotifier` -> `type fakePushNotifier struct`; `plan:internal/boardengine_test#requireOneFailedPush` -> `func requireOneFailedPush(t *testing.T, hubPath, repo string, class fabriclayout.PushFailureClass) fabriclayout.FailedPush`; `plan:internal/boardengine_test#requireNoFailedPush` -> `func requireNoFailedPush(t *testing.T, hubPath string)`; `plan:internal/boardengine_test#bareBranchTip` -> `func bareBranchTip(t *testing.T, bare, branch string) string`; `plan:internal/boardengine_test#boardEntry` -> `func boardEntry(slug string, options ...func(*boardengine.Task)) boardengine.Task`; `plan:internal/boardengine_test#withBoardTitle` -> `func withBoardTitle(title string) func(*boardengine.Task)`; `plan:internal/boardengine_test#boardFileOf` -> `func boardFileOf(tb testing.TB, entries ...boardengine.Task) string`; `plan:internal/boardengine_test#boardTitles` -> `func boardTitles(t *testing.T, dir string) map[string]string`. Name them in the fixtures bullet beside reconcileHub, with the failure fixture's boardBranch and originCommit as its methods. lym loom validate-plan --overlay accepts the lines."
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/12-board-test-moves.md:48
    summary: "Card 12 says this concern's fabric tests stop importing internal/shedrun and internal/shedengine, yet codetip_integration_test.go, the file of TestOriginCodeTip that card 4 edits, keeps both imports through plantRunTransition. That helper calls shedrun.TreeRunFileRels and marshals a shedengine.Status, the pair-push owner-guard rows share it, and no card names it"
    fix: "In card 12, add `internal/fabricengine_test#plantRunTransition` to Edit. State that it plants the transition commit with no status record, so the codec reads no machine, which card 4's TestOriginCodeTip no longer asserts, and that codetip_integration_test.go drops both shed imports. A pair-push owner-guard row that needs a planted machine moves to a package above fabric with the other moves. Unverified: which owner-guard rows read the planted machine, since their bodies were not read. lym loom validate-plan --overlay accepts the added line."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/04-origin-reads-seams.md:66
    summary: "Card 4's TestInstallRequiredSeams checks the run-lock adapter through a moving ReconcileOrigin, but tryPrimeLock calls shedrun.PrimeRunLock directly until card 11 (originreconcilemoves.go:45), so at card 4 the row passes without reaching the adapter. Once card 11 lands, it holds the same assertion as card 11's new TestReconcileOrigin_PrimeLockHeld: a held prime lock reads the prime's branches busy, and nothing moves"
    fix: "In card 4, drop the run-lock bullet (line 66) from TestInstallRequiredSeams, keeping its paths and codec checks. In card 11, add `plan:internal/fabriccli_test#TestInstallRequiredSeams` to Edit and give it that run-lock check. Remove the Create line `plan:internal/fabricengine_test#TestReconcileOrigin_PrimeLockHeld` and its Tests bullet. State that the prime-lock row of TestReconcileOrigin_Moves is deleted, its assertion held by TestInstallRequiredSeams' run-lock check through the real adapter. lym loom validate-plan --overlay accepts both card edits."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/04-origin-reads-seams.md:65
    summary: "Card 4's TestInstallRequiredSeams plants its transition 'as TestOriginCodeTip plants one', but plantRunTransition and pairFixture are private to package fabricengine_test (codetip_integration_test.go:22-39), and internal/fabriccli_test cannot reach them"
    fix: "In card 4's TestInstallRequiredSeams bullet (line 65), state the planting the test does itself in package fabriccli_test. It writes the status holding two history rows through state.UpdateJSON over shedengine.Status at shedrun.StatusFile in the pair's records worktree. It commits that with a message carrying fabriclayout.TransitionTrailerKey and fabriclayout.WarpSHATrailerKey naming the pair's code tip, then pushes and fetches. Drop the reference to TestOriginCodeTip's helper."
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/11-origin-moves-core.md:35
    summary: "Card 11 makes tryPrimeLock's os.MkdirAll path come from the process-installed RunLocks seam, which a test sets to a path of its own. Yet it keeps cmd/lym#uncontainedWriteAllowlist under Uses and claims its entry 'stays true'. The entry's reason, 'the path is shedrun's, never caller-derived' (cmd/lym/uncontainedwrite_test.go:72-73), becomes false, and the new caller-supplied write path states no bound"
    fix: "In card 11, move `cmd/lym#uncontainedWriteAllowlist` from Uses to Edit, and replace the claim at line 47. Say the originreconcilemoves.go entry's reason is restated: the path is the installed run-lock seam's, shed's prime run-lock path in every lym process (D13's installer) and a test's own hub path under the sequential rule. Bound it: only the installer and SetRunLocks set it. lym loom validate-plan --overlay accepts the move."
  - id: F11
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/10-origin-ended-core.md:36
    summary: "Cards 10 and 11 keep recordedRuns and recordedPairs decoding the run status record with fabric's own readRunStatusRows (replay.go:231-245, a JSON decode whose error fails the reconcile) and branching on whether a run's status record exists. D13 names exactly that as a rule that asks the seam's decoder for a verdict fabric defines, so a shed record decode stays in fabric"
    fix: "Choose one. (a) Add a verdict to the status codec seam, such as `StatusCodec.Recorded func(data []byte) (bool, error)`, with shed's adapter answering it. Card 10's recordedRuns and card 11's recordedPairs then read each status file's bytes through RunRecordPaths.StatusFile and ask that verdict, in place of readRunStatusRows. Card 2 adds the shed half and its test, and card 3 the field. (b) Record through the parent, as an added entry, that a presence-and-parse read of the status file is fabric's own file read and not a decode D13 moves behind the codec. Today's readRunStatusRows then stays for both."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/07-push-side-core.md:46
    summary: "Card 7 adds five outcome classes to outcomeClassOf, so the sentence in internal/fabricengine/doc.go:651-652 that lists the mapping ('done, anchor stop, auth refused, rejected, destructive refusal, transport failure or failed') becomes false. The card rewrites existing text but lists the doc under Extends"
    fix: "In card 7, move `internal/fabricengine/doc.go` from Extends to Edit, and say the outcome-class sentence of the executor section is rewritten to list the new classes. lym loom validate-plan --overlay accepts the move."
  - id: F13
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-origin-reads-seams.md:46
    summary: "RemoteOnly(hasOrigin, tips, branch) needs origin's tips before it can answer the no-origin case, while today RemoteOnlyCommits returns at the origin-URL check before CurrentBranch, CurrentSHA and the remoteBranchTips network read (remoteonly.go:14-34). A single call adds a read, or meets its error, on the no-origin path"
    fix: "In card 4's RemoteOnly bullet (line 46), say the engine asks RemoteOnly first with hasOrigin alone and nil tips. A false answer ends with no further read; only after that does the engine read the branch, the head and origin's tips and ask it again. Keep the signature."
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/10-origin-ended-core.md:45
    summary: "Card 10 puts the reads of local parts and snapshot tags in 'the under-lock phase' but does not say which runRule runs EndedTeardowns or which lock that phase takes. Today reconcileEnded takes no lock and runs after judgeAgainstOrigin has released the remote-refs locks (originreconcile.go:615-622). Card 9's OriginJudge holds those locks until its runRule returns, so running the teardowns in that rule would hold them across every teardown"
    fix: "In card 10's runner bullet (line 45), state that EndedTeardowns runs in its own runRule after OriginJudge's has returned, with a nil lock as today, and that the local-part and snapshot-tag reads are that rule's PhasePreLock reads."
---

### [BLOCKING] Cards 7 and 9 plan tests in package fabricengine that cannot import hubforge

**Location:** `_lym/plan/07-push-side-core.md:21`, and `_lym/plan/09-origin-judge-core.md:19`

**Issue:** `TestPushClassified_Snapshot` and `TestReconcileOrigin_Snapshot` are planned as tier1 tests "in package `fabricengine`" on a hubforge pair. `internal/hubforge` imports `internal/fabricengine` (for example `hub.go`'s `fabricengine.AddOptions`), so an internal test file that imports hubforge is an import cycle. None of the 68 package-`fabricengine` test files imports hubforge today, and `internal/fabricengine/doc.go`'s test-layout note sends a new test to `fabricengine_test` with `export_test.go` accessors.

### [BLOCKING] Card 7 sends a protected-branch refusal past today's retry and judgment

**Location:** `_lym/plan/07-push-side-core.md:108`

**Issue:** `mapPlainPushError` wraps a declining hook's refusal in `ErrPushRejected` (`internal/gitrepo/push.go:292`). So today a protected push goes through `settleRejection`, whose retry fetches and pushes once more, and then through `classifyFailure`'s rejection case to `failRejection` (`pushclassified.go:164-176`, `639-640`, `664-690`). Card 7 orders `OutcomeProtected` ahead of rejected and records protected as `other` at once, so neither the retry nor the judgment read runs. The failed-push record's error loses the judgment's suffix, and an origin that is ahead reads `other` in place of `not_fast_forward`. `AProtectedBranchRefusalRecordsTheRejectionClassAndIsNotRetried` asserts only the class and the zero sleep count, so it does not catch this.

### [BLOCKING] Card 7 pushes an exhausted push again as if it were a transport failure

**Location:** `_lym/plan/07-push-side-core.md:93`

**Issue:** `retryTransportPush` loops only while `gitexec.IsTransportFailure` holds (`pushclassified.go:205-221`). An exhausted inner retry wraps `ErrPushRetriesExhausted` over a `(failed)` or `cannot lock ref` refusal, which `IsTransportFailure` does not class as transport. `classifyFailure` records it as transport and pushes nothing again. Card 7's mapping folds it into `OutcomeTransportFailure`, which the card's transport rule pushes again after each delay. Each of those rounds runs gitrepo's own bounded retry.

### [MEDIUM] An unset run-records seam surfaces mid-push, not before acting

**Location:** `_lym/plan/04-origin-reads-seams.md:40`

**Issue:** Once `originStatusAtTip` reads through `originStatusBytesAtTip`, the refusal reaches `ReadUpstreamTakeover` (`takeover.go:95`) inside `retryWeftPush`'s `takeoverStop` (`pushclassified.go:984-991`). That runs after a rejected first push, so the refusal becomes a retry error that `settleRejection` joins to the rejection and records as `other`. No card of this part runs `refuseUnsuppliedSeams` on the push verbs, while card 4 calls it the refusal every verb of the part runs before it reads or acts.

### [MEDIUM] Card 3's search misses the indirect seam readers

**Location:** `_lym/plan/03-required-seam-installer.md:55`

**Issue:** The search names three direct readers, in fabricengine's tests only. Card 4's move adds `OriginParentRunStatus`, `OriginBattenStatus` and `ReadUpstreamTakeover`, the last reached by every push that retries a records rejection. `ReconcileOrigin` is called from a dozen packages outside fabric (`fabriccli/remove.go`, `beamcli/dobbydeps.go`, `orchcli/start.go`, `battencli`, `loomcli`, `cleanupcli`, `hibernatecli` and `pairteardown`), whose tests the search never reads. The card also omits card 11 from the cards that move shed calls onto the seams.

### [MEDIUM] The tests card 12 moves to boardengine_test lose their fixtures

**Location:** `_lym/plan/12-board-test-moves.md:12`

**Issue:** The moved `pushclasses` rows call helpers private to `fabricengine_test`: `newFailureFixture`, `recordsFixture`, `fakePushNotifier`, the failed-push assertions and the board-file builders (`pushclasses_integration_test.go:45-83`, plus the helpers the `ABoltPushRejectedOverAnUnrelatedOriginEntryIsReplayedAndLands` row calls). The card recreates only the reconcile fixtures.

### [MEDIUM] codetip_integration_test.go keeps both shed imports

**Location:** `_lym/plan/12-board-test-moves.md:48`

**Issue:** `plantRunTransition` (`codetip_integration_test.go:15-39`) builds the planted status with `shedrun.TreeRunFileRels` and `shedengine.Status`. It belongs to the originreads concern, which card 4 edits, and no card names it, so this concern's fabric tests keep a shed import that card 12's Intent says they drop.

### [MEDIUM] Card 4's run-lock adapter check runs before the seam is read, and then duplicates card 11's test

**Location:** `_lym/plan/04-origin-reads-seams.md:66`

**Issue:** At card 4, `tryPrimeLock` still calls `shedrun.PrimeRunLock` itself (`originreconcilemoves.go:44-45`), so the check proves nothing about the adapter. From card 11 on, both it and `TestReconcileOrigin_PrimeLockHeld` assert that a held prime lock reads the prime's branches busy with nothing moved. That is one behaviour tested twice.

### [MEDIUM] Card 4 cites a planting helper its test cannot reach

**Location:** `_lym/plan/04-origin-reads-seams.md:65`

**Issue:** `plantRunTransition` and `pairFixture` live in package `fabricengine_test`. `TestInstallRequiredSeams` lives in `internal/fabriccli_test` and cannot import them.

### [MEDIUM] Card 11 falsifies the uncontained-write allowlist's reason

**Location:** `_lym/plan/11-origin-moves-core.md:35`

**Issue:** The entry for `originreconcilemoves.go` at `cmd/lym/uncontainedwrite_test.go:72-73` justifies `tryPrimeLock`'s `MkdirAll` because "the path is shedrun's, never caller-derived". After card 11 the path is whatever the installed `RunLocks` seam returns, which a test sets to its own hub path. D7 moves an allowlist that names a moved symbol with that symbol, and the attack-surface rule wants the new caller-supplied path bounded.

### [MEDIUM] Recorded-run reads still decode shed's record in fabric

**Location:** `_lym/plan/10-origin-ended-core.md:36`

**Issue:** `recordedRuns` (`originreconcileended.go:55`) and `recordedPairs` (`originreconcilemoves.go:549`) call `readRunStatusRows`. That function decodes the status record into fabric's own `runStatusRows` and fails on a decode error (`replay.go:231-245`). Cards 10 and 11 swap only the path source. D13's bullet on rules that branch on a record names "whether a run's status row exists" as a verdict the seam's decoder gives.

### [LOW] Card 7 rewrites fabricengine's doc.go under Extends

**Location:** `_lym/plan/07-push-side-core.md:46`

**Issue:** `internal/fabricengine/doc.go:651-652` lists `outcomeClassOf`'s classes, and card 7 changes that list. A rewrite of existing text is an Edit, not an Extends.

### [LOW] RemoteOnly's signature needs a read today's code skips

**Location:** `_lym/plan/04-origin-reads-seams.md:46`

**Issue:** `RemoteOnlyCommits` returns at its origin-URL check before any other read (`remoteonly.go:14-34`). One call over `hasOrigin` and `tips` needs the tips first, so the no-origin path would gain the `remoteBranchTips` read unless the card says the engine asks twice.

### [LOW] Card 10 leaves the teardown rule's run and lock unnamed

**Location:** `_lym/plan/10-origin-ended-core.md:45`

**Issue:** Card 9 keeps both remote-refs locks held until `OriginJudge`'s `runRule` returns. Card 10 does not say whether `EndedTeardowns` runs in that rule or in its own, nor which lock its "under-lock phase" takes. Today the teardowns run with no lock, after `judgeAgainstOrigin` has released the remote-refs locks.

## Rejected

- guard-reach F6 (card 9 refuses an unset `SeamRunLocks` for every moving hub scope): D13's acceptance criterion puts the refusal in the pre-lock phase, where only the scope is known, and a moving hub scope is the one scope that may take the prime lock. Refusing there is the only way to meet both rules, so it is no defect.
- Handler: `waitRecordsLocks`' push-lock poll stays an engine loop with its own sleep (card 11). D4 keeps every lock taken and held by the engine exactly where it is today, and the poll is a lock acquisition, so it is not a retry the core must ask for as a wait action.
- Handler: cards 4, 5, 6 and 8 move rules as verdict helpers the engine calls, not as `runRule` answers (focus entry 5). Those verbs make no mid-flow branch on an action's answer, and D3's action-list shape binds where a rule branches mid-flow. The acceptance table's verdict rows cover them, so this is no defect.
- Handler: the fallback test for ref storage other than `files` (focus entry 8). The foundation's `TestSnapshotReader_RefStorage` holds it, and card 7's per-verb readers make today's `gitrepo` calls, so no new reader needs a fallback test of its own.
- Handler: card 7's ended-entry tracking-ref delete outside `destroy.go`. Today's code deletes it through `DeleteRemoteTrackingRef` with no guard, and D4 keeps the guards exactly where they are, so it is not a new bypass.

## Focus departures

- "Test moves and cuts (cards 4-12): check each deleted row against the core row said to hold its assertion…": this was driven for the rows behind the findings above: the protected and exhausted-push rows of `TestPushFailureClasses`, the prime-lock rows and the moved Bolt rows. It was not driven row by row for every deleted row of `TestFailedPushRecord`, `TestRefusedPushStands`, `TestRunCommitsAhead`, `TestTipsOnOrigin`, `TestDescribePushLockHolder` and `TestReconcileOrigin_Judge`. The round's reading budget went to the blocking findings, and the commit message is where each such cut is named.
- "Card 12's completeness…": this was driven for the shed and board imports and for the moved tests' fixtures (F6, F7). The reviewer did not read every kept row of `TestReconcileOrigin_Moves` and `TestReconcileOrigin_Judge` to check that it still reaches its branch once it records no run. The guard-reach fork traced only `TestBoltReplayers`' no-replayer row.
