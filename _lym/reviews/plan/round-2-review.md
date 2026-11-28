---
verdict: BLOCKING
findings:
  - id: F15
    severity: BLOCKING
    class: scope
    location: "_lym/plan/07-push-side-core.md:97"
    summary: "Card 7's new pre-lock refusal in coalescePushSides breaks TestCoalescePushRequest_CoveredCallDefers and TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock (internal/fabricengine/pushdeferral_integration_test.go, package fabricengine). Both push plain repos with no hub, and package fabricengine cannot import hubforge, so no seam is installed. No card names them, and card 3 says the package-fabricengine hits of its search reach no seam read"
    fix: "In card 7, add `internal/fabricengine#TestCoalescePushRequest_CoveredCallDefers` and `internal/fabricengine#TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock` to Edit. State in the Intent that each sets the run-records paths seam itself through SetRunRecordPaths, with paths under its own temp repos, restores it before it returns and stays free of t.Parallel, since internal/hubforge imports internal/fabricengine and an in-package test cannot take hubforge's install. All three subtests of the first test call CoalescePushRequest and need it. In card 3, replace the sentence on the package-fabricengine hits: the search also lists pushdeferral_integration_test.go, which reaches coalescePushSides with no hub and which card 7 handles, and commit_partial_integration_test.go, which only stubs the detached spawn. Drop the claim that the search finds no such file. validate-plan --overlay accepts the two Edit lines."
  - id: F16
    severity: BLOCKING
    class: decision
    location: "_lym/plan/07-push-side-core.md:90"
    summary: "Card 7 treats every protected-branch refusal as an unsettled rejection, but the refusal of resetUnnamedTip's leased reset wraps no ErrPushRejected. Today it records `other` with the reset's own error and reads no judgment facts. Planned, it fetches and, with origin ahead, records not_fast_forward, which changes class, text and git process count against D5"
    fix: "Choose one. (a) Keep today's path: add the Create line `plan:internal/fabriccore#OutcomeProtectedUnrejected` -> `const OutcomeProtectedUnrejected OutcomeClass = \"protected_unrejected\"`. outcomeClassOf maps to it a protected refusal that does not wrap gitrepo.ErrPushRejected, and keeps OutcomeProtected for one that does. That retry outcome stops the loop, its error is not joined to the rejection, and JudgePush records `other` with that error and asks no FieldPushJudgment. Add a TestPushSide row and a TestOutcomeClassOf row. The added entry `A protected-branch refusal keeps the rejection path` says a protected retry outcome reads the judgment facts, so the parent narrows it to a refusal of the push itself. validate-plan --overlay accepts the Create line. (b) Have the parent record that the reset's protected refusal may take the judgment path, and say so in card 7."
  - id: F17
    severity: BLOCKING
    class: design
    location: "_lym/plan/12-board-test-moves.md:28"
    summary: "Card 12 creates statusJSON in boardengine_test, a hand-writer of shed's run record as a map literal, and restates it for the moved rows. D12 bars feeding fabric a hand-written shed run state in a receiving package, and PATTERN-test-stand-ins bars adding such a stand-in"
    fix: "Remove the Create line `plan:internal/boardengine_test#statusJSON` and its mention in the fixtures bullet. Re-sign the Create line of writeRunStatus to `func writeRunStatus(tb testing.TB, prime *lymcwd.Location, slug, producer string, history ...string)`, writing shed's Status and HistoryEntry types through state.UpdateJSON. State that every moved row that calls statusJSON today (APrimeWeftPushRejectedAfterThisMachinesFinalizeKeepsTheLandingsCommits writes a pair record and commits a prime record with it) writes each record that way at shedrun's path and commits the written file. validate-plan --overlay accepts both line changes."
  - id: F18
    severity: BLOCKING
    class: design
    location: "_lym/plan/12-board-test-moves.md:101"
    summary: "Card 12 restates in boardengine_test helpers that stay defined and used in fabricengine_test, among them a notifier fake, and wraps hubforge in fixture types (failureFixture, pairFixture, pushPair, reconcileHub). PATTERN-test-isolation bars duplicating shared support across packages and bars a test package's own hubforge fixture type; PATTERN-test-stand-ins bars adding another notifier fake"
    fix: "Remove the Create lines of reconcileHub, newReconcileHub, failureFixture, newFailureFixture, pairFixture, newPairFixture, pushPair, newPushPair, fakePushNotifier, bareBranchTip, originHasBranch, requireOneFailedPush, requireNoFailedPush, failedPushesOf and countingBackoff in boardengine_test. State that moved rows take a *hubforge.Hub and read its accessors, with any former method a plain function over the hub. They read bare tips and branches through gitkit.RevParse and gitkit.BranchExists. The helpers both packages need that use neither git nor hubforge go into one new kit, with the Create lines `plan:internal/testkit/pushkit#RequireOneFailedPush` -> `func RequireOneFailedPush(t *testing.T, hubPath, repo string, class fabriclayout.PushFailureClass) fabriclayout.FailedPush`, `plan:internal/testkit/pushkit#RequireNoFailedPush` -> `func RequireNoFailedPush(t *testing.T, hubPath string)` and `plan:internal/testkit/pushkit#CountingBackoff` -> `func CountingBackoff(sleeps *int, onSleep func(call int)) func(time.Duration)`, and fabric's copies go with their callers retargeted. A records write-lock holder, if a moved row needs it, joins that kit under a name with no side word. Moved rows pass SyncOptions with no Notifier and drop their mail-line assertions, which TestMergeFailedPush's notify rows hold. For tagsOf, landOn, originHasBranch and boardFileOf, whose fabric definitions sit in other concerns' files (D9), the moved rows call gitkit or boardkit exports instead of restating them. validate-plan --overlay accepts the removals and the three kit Create lines. Unverified: what the installed notifier does in a test binary that installs none, and whether gitkit and boardkit already export every read and board write these rows need; where one does not, the fixer adds it there."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
    answer: "Settled by the added entry `A protected-branch refusal keeps the rejection path`, which card 7 carries. F16 is a separate path the card's wording takes in."
  - id: F3
    status: resolved
  - id: F4
    status: resolved
    answer: "Settled by the added entry `Push verbs refuse an unset run record seam before the push lock`, which cards 4, 7 and 8 carry."
  - id: F5
    status: resolved
    answer: "The search is replaced as asked. Its reported result is wrong, which F15 raises."
  - id: F6
    status: resolved
    answer: "The moved rows' helpers are now named. How they are added breaks other rules, raised as F17 and F18."
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
    answer: "Settled by the added entry `Fabric asks a status codec verdict instead of decoding shed records`, which cards 2, 3, 9, 10 and 11 carry."
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
---

# Plan review, round 2

The round-1 fixes settle the listed findings.
Four defects sit on changed hunks.
All four come from reading the code and the plan; none was run, since the planned code does not exist yet.

### [BLOCKING] Two in-package push tests meet the new seam refusal

**Location:** `_lym/plan/07-push-side-core.md:97`, `_lym/plan/03-required-seam-installer.md:61`

**Issue:** Card 7 makes `coalescePushSides` refuse an unset run-records paths seam before the push lock.
`internal/fabricengine/pushdeferral_integration_test.go` is in package `fabricengine`
and calls `CoalescePushRequest` and `CoalescePushBothAt` over `newChangedPair`'s plain repos.
`hubforge.InstallRequiredSeams` is out of its reach by the import cycle,
and its serial tests run before the external package's tests build any hub.
Each call that expects nil or a gate exit gets the refusal instead.
Card 3's own search lists this file, but the card names five other files as the package's only hits.

### [BLOCKING] The reset's protected refusal changes its record

**Location:** `_lym/plan/07-push-side-core.md:90`, `_lym/plan/07-push-side-core.md:115`

**Issue:** `repushOverOriginContained` calls `resetUnnamedTip`, which moves origin's branch through `updateRemoteBranch`.
`gitrepo.UpdateRemoteBranchLeased` returns a hook or ruleset refusal as a wrapped git error, never `ErrPushRejected`.
Today `rejectionRetryKind` names it as protected and not retryable, so `settleRejection` returns it unjoined,
and `classifyFailure` matches no class, which records `other` with that error.
Card 7 maps it to `OutcomeProtected` like a rejected push, and sends every protected refusal to the judgment read.
The reset runs only when origin holds commits the branch lacks, so that read answers `not_fast_forward`.

### [BLOCKING] `statusJSON` is a hand-written shed record

**Location:** `_lym/plan/12-board-test-moves.md:28`

**Issue:** The helper (`internal/fabricengine/replay_integration_test.go`) marshals a map literal of shed's JSON keys.
Card 12 restates it in `boardengine_test`, the receiving package, although the same card already writes records through shed's types.

### [BLOCKING] Restated helpers and hubforge wrapper types

**Location:** `_lym/plan/12-board-test-moves.md:101`

**Issue:** `requireOneFailedPush`, `requireNoFailedPush`, `bareBranchTip`, `countingBackoff`, `fakePushNotifier`
and others keep users in `internal/fabricengine` after the move, so the card creates a second copy of each.
`PATTERN-test-isolation` puts shared support in one kit and says no test package wraps `hubforge` in its own fixture type.

## Rejected

- Focus 1, other error kinds: auth refusal, transport, exhausted retries, plain rejection, destructive refusal,
  anchor stop, takeover and deferral keep today's loops and classes; only F16 differs.
- Focus 2: the push verbs, `OriginRunStatus`, `OriginCodeTip` and `ReconcileOrigin` refuse before they act.
  The handover readers' later refusal is the parent's ruling.
- Focus 4 and 6, CLI-driven callers: tests that reach a push through `lym fabric push` or `sync` build hubforge hubs,
  and the role-guard rows assert only the guard's own marker.
- Focus 7: `finalizeEntered` still decodes run records in `replay.go`, which is the replay concern's file (D9).
- Focus 8: no planned `boardengine_test` name collides with an existing one,
  and `codetip_integration_test.go` imports shed only for `plantRunTransition`.
- Focus 10: with no run recorded, the kept move and judge rows still reach their branch verdicts,
  since the live probe, the locks and the fast-forward do not depend on a run record.

## Focus departures

- "Cards 4-12, deleted test rows of TestFailedPushRecord, TestRefusedPushStands, TestRunCommitsAhead, TestTipsOnOrigin,
  TestDescribePushLockHolder and TestReconcileOrigin_Judge: is each deleted row's assertion held by the core row or test
  the card names for it?"
  Not checked row by row.
  The deletion lists and their named holders are unchanged since round 1, and this incremental round takes new findings
  only on changed hunks.
  The one changed deletion, card 11's prime-lock row, is held by the run-lock check it names for the busy verdict.
