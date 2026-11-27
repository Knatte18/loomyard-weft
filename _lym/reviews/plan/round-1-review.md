---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/09-records-hold.md#Intent
    summary: "The records hold is applied only inside `pairteardown.PushRun`, but `lym fabric push` in a task worktree calls `fabricengine.PushPairAnchored` directly with no run slug and no hold (internal/fabriccli/weft_verbs.go:314), so it sends the held records commit to origin while the landing entry stands, against D1's last sentence and D5's rule that the hold binds every push of the run's pair records."
    fix: "In card 9, move the hold read into `PushPairAnchored` itself: when the request's `RecordsHold` is empty and `l` is a task pair (not the prime), it reads `pushledger.HeldRecordsCommit` for the pair's slug and applies it, so `lym fabric push` and its detached bypass (`fabricengine.DetachedPush`) are held like every `PushRun` caller. Keep `CodeCeiling` set only by `PushRun`, so the Bound's beam-parent `lym fabric push` still pushes the beam's code side whole. Add a row to `TestPushRun_StopsShortOfAHeldRecordsCommit` (tier1) that runs `lym fabric push` in the held pair's task worktree and asserts origin's records branch lacks the held commit."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/11-parent-pusher.md#Intent
    summary: "Card 11 step 5 pushes a beam parent through 'the beam pair's `PushRun` with that tip as its code ceiling', but no card gives `pairteardown.RunPush` a ceiling input and card 9 has `PushRun` derive `CodeCeiling` itself as the first parent of the oldest landing's tip, so the parent push never sends a landing commit and no beam landing ever settles."
    fix: "In card 9, add `internal/pairteardown#RunPush` to Edit with a `CodeCeiling` field; `PushRun` passes a set `RunPush.CodeCeiling` through to `PairPush.CodeCeiling` in place of its derived ceiling, and derives one only when the field is empty. In card 11, add `internal/pairteardown#RunPush` and `internal/pairteardown#PushRun` to Uses, and have step 5 set `RunPush.CodeCeiling` to the judged tip. Add a `TestPushRun_StopsShortOfAHeldRecordsCommit` row: a beam pair holding a landing, pushed with `CodeCeiling` set to the landing tip, ends with origin's code branch at that tip and its records branch pushed first."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/19-teardown-hands-to-dobby.md#Intent
    summary: "Ledger removals drop entries that still hold landings, against D3's 'a parent target's entry stays while any of its landing entries stands'. Card 19's `RemovePair` removes the removed pair's entry unconditionally, so removing a beam pair deletes its children's landings, their pending `done` writes and holds. Card 6's `PruneLedger` drops any entry whose `Worktree` is gone. No card says who writes `Worktree` and `Slug` on the landing-key and beam-key entries, which `PruneLedger` and `HeldRecordsCommit` read."
    fix: "In card 19, make the teardown composite's check refuse also while the removed pair's own ledger key holds landings of other runs, naming the owed parent push as its way forward, and have `RemovePair` remove only an entry holding no landings. In card 6, have `PruneLedger` keep every entry holding landings and skip the board and landing keys. In card 11, have `RecordLanding` write the entry's `Worktree` (the parent's code worktree) and, for a beam parent, `Slug` (the beam's slug); have step 2's drop write `Slug` (the run's slug) and `Worktree` on the pair entry it creates for `HeldRecords`. Add rows: in `TestPruneLedger`, a gone-worktree entry holding a landing stays; in `TestTeardown_CallSequence`, removing a beam pair whose key holds a child's landing refuses."
  - id: F4
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/05-pusher-interval.md#Intent
    summary: "A pair whose records branch is ahead only by held commits never reads level. `RunCommitsAhead` counts the held commit, card 6 makes `Ahead` true while landings stand, and the held push stops short at `<hold>^`. So its pusher runs a no-op push and its run-scope reconcile once per interval with no end. Until `lym fabric remove`, a dropped landing's `HeldRecords` entry keeps D3's owed period open with no all-clear and keeps D4's minutes counting, and the `TestRun` row 'a level pass keeping the entry while it holds a landing' pins a state that cannot occur."
    fix: "In card 5, give `Deps.Ahead` of a target whose entry holds a hold or `HeldRecords` the read 'commits past origin up to `<hold>^`' (through `pushledger.HeldRecordsCommit`), so a pair ahead only by held commits reads level and its pusher exits keeping the entry. A landing-holding parent target waits on its push-now mark, not the interval, between parent passes. In card 20, list an entry that holds only `HeldRecords` as held, not owed: it does not open or hold an owed period and does not block the all-clear. In card 21, leave it out of the bar's minutes. Replace the unreachable `TestRun` row with 'a pair ahead only by its held commit reads level, exits after one pass and keeps its entry', and add a `TestPushSummary` row for an all-clear with a `HeldRecords` entry standing."
  - id: F5
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/11-parent-pusher.md#Intent
    summary: "Step 4's refusal of a tip carrying a landing commit no landing records names `lym loom resume` in that run's task worktree. That way forward works only while the run is still at Finalize: internal/loomcli/resume.go:88 answers 'the run is done; there is nothing to resume'. The refusal is also reached for a run past Finalize: after a lost ledger, after F3's removals, and after a landing that step 2 dropped with other local commits on it."
    fix: "In card 11, have step 4 skip a landing commit whose run's landing step 2 dropped (that case already carries step 2's rebase refusal). For a run past Finalize, re-record the landing from the trailer commit, as F6's fix describes, instead of refusing. Keep the `lym loom resume` way forward only for a run whose loom status is not done, which is read through the run's status file. Add `TestParentPush` rows: an unrecorded landing commit of a done run is re-recorded and pushed, and one of a run still at Finalize is refused naming `lym loom resume`."
  - id: F6
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/11-parent-pusher.md#Intent
    summary: "PATTERN-lym-files: 'Nothing a run, the orch or a landing depends on lives only under `.lym`', and an absent cache reads as empty and refuses nothing. The plan keeps a landing's pending `done` write and its records hold only in the machine-local ledger under `.lym`. If the ledger is lost, the hold is gone, so the pair pusher pushes the run's records and Worktree-Teardown finds both tips on origin and no entry, then removes the pair. The board never gets `done`, and origin reads the run's records while its parent lacks the work: the state D5 and D13 forbid."
    fix: "Make the landing entries rebuildable from git, so the ledger is a cache. In card 11, `settleParent` and `SettleRefused` treat every `Lym-Landing` trailer commit that `fabricengine.LandingCommits` finds past origin, whose run has no landing entry and whose board entry is not done, as a landing to re-record through `RecordLanding`, holding the run's pair records tip at the time of the re-record. In card 19, `Teardown`'s `landingStands` and the Worktree-Teardown `Level` read also answer true for such an unrecorded trailer commit of the run on its parent. In card 9, `HeldRecordsCommit` callers re-record before they push. Add a `TestParentPush` row that deletes the ledger directory after a landing and asserts the next pass re-records the entry, keeps the records commit off origin and makes the `done` write only after the parent push lands."
  - id: F7
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/11-parent-pusher.md#Intent
    summary: "PATTERN-test-stand-ins forbids a stand-in for a repository module ('when another module fills the seam in production, the fake simulates that module'), and 'no change adds a stand-in this rule forbids'. The plan's key tests run over such fakes and reach D5's and D8's criteria only through them. Card 11's `TestParentPush` fakes `ParentDeps`, which `pusherDeps` fills from `fabricengine`, `boardengine` and `pairteardown`. Card 13's rows fake `ParentPushState`, card 17's `TestPushOwedForCatchUp` fakes `pairteardown.PushRun` and `runpusher.Start` errors, and card 19's rows fake `landingStands`, which `pushledger` fills. Card 12's real `landLocally` wiring has no test."
    fix: "In card 11, turn `TestParentPush` into a tier1 test in `internal/runpusher` over a hub fixture with a bare origin. It drives the real `fabricengine` parent steps, the real `boardengine` `done` write and the real refusal record writer through the `pusherDeps` wiring, and stands in only for git transport through the shared `gitkit` boundary. Keep in tier0 only rows whose seams the module under test fills itself. In card 17, `TestPushOwedForCatchUp` holds the real push lock (`fabricengine`'s lock path) over a pair fixture instead of faking `ErrPushLockBusy`. In card 19, write the landing entry through `pushledger.Update` instead of a fake `landingStands`. In card 13, drive `ParentPushState` through the real `landingDeps` wiring over a ledger entry. In card 12, add a tier1 `landLocally` test over a hub fixture: it commits the status file, writes the landing entry with that records commit, and starts the parent's and the pair's pushers. State each moved test's seconds against the tier budget."
  - id: F8
    severity: BLOCKING
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/04-push-ledger.md#Create
    summary: "The new test package `internal/pushledger` (ledger_test.go and card 21's `TestSyncStatus`) has no `TestMain` calling `tmuxkit.Main`. PATTERN-test-isolation's tier0 guard `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain` (cmd/lym/tmuxisolation_test.go:266) fails such a package."
    fix: "In card 4, add `internal/pushledger/testmain_test.go` to Create, and add the Intent line '`internal/pushledger/testmain_test.go` is untagged support: its `TestMain` runs the package through `tmuxkit.Main`.' after the line that gives ledger_test.go's tier."
  - id: F9
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lym/plan/12-finalize-local-landing.md#Delete
    summary: "Card 12 deletes `loomcli#parentMergeArchiver`, the only production caller of `fabricengine.ArchiveParentMerge` (internal/loomcli/landingdeps.go:928; Finalize's own seam goes with `ArchiveParentMerge` in `Deps`). The function then becomes test-only code, which PATTERN-test-only-code forbids, and its test and doc line stay."
    fix: "In card 12, add `internal/fabricengine#ArchiveParentMerge` and `internal/fabricengine_test#TestArchiveParentMerge` to Delete. Add `internal/fabricengine/doc.go` to Edit, with an Intent line that drops its `ArchiveParentMerge` line. So that `uses-later-target` passes, remove `internal/fabricengine/doc.go` from the Uses of cards 8, 9 and 10, and drop card 8's then-empty `**Uses:**` field."
  - id: F10
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/09-records-hold.md#Intent
    summary: "`CodeCeiling` changes what `PushBeam` pushes. While landings stand on a beam, `lym beam pr` and its edit path push the beam only below its landing commits, yet record and open the pull request against `pair.HeadSHA()`, a head origin's beam branch lacks (internal/beamcli/deps.go:371-378, internal/beamcli/pr.go:81 and :175). No card names `PushBeam`, `pr.go` or internal/beamcli/doc.go:103,108, which say `lym beam pr` pushes the beam branch."
    fix: "In card 9, add `internal/beamcli#hubLandDeps` (which holds the `PushBeam` closure) and `internal/beamcli/doc.go` to Edit. `PushBeam` refuses while the beam's own ledger key holds landings, with the way forward 'wait for the beam's parent push, or run the step its refused-push record names, then rerun `lym beam pr`'. Add a tier1 row in the `lym beam pr` tests, and add one line to beamcli's doc. To pass `uses-later-target`, remove `internal/beamcli/doc.go` from card 6's Uses."
  - id: F11
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/07-spawns-start-the-pusher.md#Edit
    summary: "Card 7 registers a pair's spawn under the hub owner with the push role. `TestStartDetachedPush_RegistersTheChild`'s row `PairPushOwnsUnderItsSlug` (internal/fabricengine/spawn_linux_test.go:38) asserts owner `task`, so it fails, and the card does not list that test."
    fix: "In card 7, add `internal/fabricengine#TestStartDetachedPush_RegistersTheChild` to Edit. Add an Intent line saying its pair row now expects `runproc.HubOwner` and the push role, and that it spawns the pusher's command line."
  - id: F12
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-driver-wait-readers.md#Uses
    summary: "internal/dobbyengine/doc.go's run-watch section defines a stop as any state other than running, and has no per-tick push summary. Card 3 makes a `Waiting` run owe no line and a `WaitDied` run one stop, and card 20 adds the summary step. Both cards list the doc only under Uses, so neither updates it."
    fix: "In cards 3 and 20, move `internal/dobbyengine/doc.go` from Uses to Edit. Card 3 adds the waiting and died-driver rule to the run-watch section, and card 20 adds a paragraph on the prime's push summary and all-clear."
  - id: F13
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/19-teardown-hands-to-dobby.md#Uses
    summary: "internal/pairteardown/doc.go:168-174 says `PushRun` pushes the run's records whole and `HandOver` removes the local pair unless it runs inside it. Card 9 adds the records hold, and card 19 adds the `ErrLandingStands` refusal, the hand-over's kept pair and `RemovePair`'s ledger removal. Both cards list the doc only under Uses."
    fix: "In cards 9 and 19, move `internal/pairteardown/doc.go` from Uses to Edit. Card 9 documents the hold. Card 19 documents the landing refusal, the kept pair on a hand-over and the ledger removal. To pass `uses-later-target`, remove `internal/pairteardown/doc.go` from the Uses of cards 5 and 15."
  - id: F14
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/12-finalize-local-landing.md#Intent
    summary: "`LandLocally` commits the run's status records first and records the landing entry holding that commit afterwards. A pair pusher that is already running (each earlier transition starts one) can push the new records commit in that window, and a kill between the two leaves the commit unheld. The resume then records a hold on a commit origin may already have, against D5's rule that origin never reads the run's pair records before its parent holds the work."
    fix: "In card 12, have `landLocally` take the pair's pusher lock (`runpusher.LockPath` of the pair's target, waiting while it is held) across the status commit and `RecordLanding`, so no pair push runs between them. Or write the landing entry before the commit with the hold set to the pre-commit records tip (card 9 then pushes up to and including that tip) and fill `RecordsCommit` after the commit. State which. Add a row to the `landLocally` test from F7: a pair pusher started between the commit and the record pushes nothing past the pre-landing records tip."
  - id: F15
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/07-spawns-start-the-pusher.md#Intent
    summary: "D1's second criterion asks a test to pin that a commit through `lym board sync`, `lym fabric sync` and `lym fabric add` each starts or wakes its target's pusher. Card 7's tests reach only `lym fabric sync`, through `Fabric.commit`, and `boardengine.spawnSync`. No test drives `lym fabric add`, and no test drives a `lym board sync` caller's commit."
    fix: "In card 7, add tier1 rows to `TestSpawnStatusPush` that run `lym fabric add` with `spawnDetachedPushFn` recording and assert the recorded line is the pair pusher's with `--push-now`. Add a row driving a `boardengine.SpawnSync` caller, such as heddle's board sync, that asserts the `--board --push-now` line. State the seconds each adds."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/23-pattern-docs.md#Prosa
    summary: "pattern/PATTERN-shed-driver.md:37 says the driver ends in the tackler, a re-step under the repair cap or a hand-back. Card 2 re-steps a network-class stop with no cap, and card 23 does not list that background."
    fix: "In card 23, add `pattern/PATTERN-shed-driver.md` to Prosa, with an Intent line: a `git-transport` or `github-api` stop is re-stepped with no cap, waiting from 15 s doubling to `fabric.push_ceiling_s`, under a wait record."
  - id: F17
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/19-teardown-hands-to-dobby.md#Intent
    summary: "With pushes on, `lym batten cancel` commits its cancel intent and spawns its push (internal/battencli/cancel.go:380-390) before `EndForCancel`, where card 19 puts the landing refusal, so where that refusal is reached it comes after a mutation, against D13. The parent-holds probe reads origin's parent (internal/battencli/cancelcmd.go:482), which lacks the work while the landing push is owed."
    fix: "In card 19, add `internal/battencli#endRunPhase` (internal/battencli/cancel.go:300, which calls `WriteIntent`) to Edit. It reads the run's standing landing entry through `pushledger` before `WriteIntent` and refuses with `ErrLandingStands`'s way forward. Add a `battencli` cancel row asserting that no intent is committed."
  - id: F18
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/00-overview.md:66
    summary: "Shared Decisions: 'a landing entry is the hold and the pending done write' carries no `Applies to:` line, while 'a landing commit carries its run in a trailer' carries two. The second, 'cards 4, 9, 11, 12, 15, 16, 17 and 19', belongs to the landing-entry decision."
    fix: "In 00-overview.md, move the line '- **Applies to:** cards 4, 9, 11, 12, 15, 16, 17 and 19' from under the trailer decision to the end of the landing-entry decision, after its Rationale. If the gate refuses an edit outside the Card Index, defer this finding naming that gate rule."
  - id: F19
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/04-push-ledger.md#Intent
    summary: "`clear.json` is written only when the last entry goes, so on a hub whose first owed entry comes before any clear time is written, `LastClear` reads zero. Card 21's bar minutes and card 20's owed period then count from the zero time, not from the time the hub last had no owed target (D3, D4)."
    fix: "In card 4, have `Update` also write `clear.json` with `now` when it adds the first entry and no clear time stands, under the same lock. Add a ledger_test.go row: a first entry on an empty hub sets the clear time to that `now`."
---

### [BLOCKING] `lym fabric push` bypasses the records hold

**Location:** `_lym/plan/09-records-hold.md#Intent`

**Issue:** Card 9 puts `RecordsHold` only in `pairteardown.PushRun`.
`lym fabric push` pushes through `fabricengine.PushPairAnchored` directly (`internal/fabriccli/weft_verbs.go:314`), with no `RunSlug`, and so does its detached bypass through `fabricengine.DetachedPush` (`weft_verbs.go:280`).
In a task worktree whose landing entry stands, the verb sends the held records commit and everything after it.
D1 says no synchronous push of a run's pair records sends that commit, and D5 says the hold binds every push of them.
The card's Bound covers only a beam's worktree.
Both the affected-set lens and the guard-reach lens raised this; I confirmed the call site.

### [BLOCKING] The beam parent push cannot pass the judged tip as its ceiling

**Location:** `_lym/plan/11-parent-pusher.md#Intent`

**Issue:** Card 11 step 5 hands the judged tip to "the beam pair's `PushRun` … as its code ceiling".
`pairteardown.RunPush` (`internal/pairteardown/handover.go:22-37`) has no ceiling field, and no card adds one.
Card 9 has `PushRun` derive `CodeCeiling` as the first parent of the oldest landing's tip.
So every beam parent push stops below the first landing commit.
Origin's beam branch never takes a landing, step 1 never settles one, and a beam's children never get their `done` writes.

### [BLOCKING] Ledger removals drop entries that hold landings

**Location:** `_lym/plan/19-teardown-hands-to-dobby.md#Intent`

**Issue:** Card 19's check reads only the removed run's own landings, and then `RemovePair` removes the removed pair's entry.
For a beam pair, that entry holds its children's landings, so `lym fabric remove <beam> --force` drops their pending `done` writes and releases their holds.
Card 6's `PruneLedger` drops any entry whose `Worktree` is gone, with no exception for landings.
No card says who writes `Worktree` and `Slug` on the landing-key and beam-key entries, or on the pair entry card 11's drop creates for `HeldRecords`.
`HeldRecordsCommit` finds that entry by `Slug`.
D3 keeps a parent target's entry while any of its landings stands.

### [BLOCKING] A pair ahead only by held commits pushes forever and blocks the all-clear

**Location:** `_lym/plan/05-pusher-interval.md#Intent`

**Issue:** `RunCommitsAhead` counts the held commit, and card 9's push stops at `<hold>^`.
So a held pair stays ahead after every push.
Card 6 also keeps `Ahead` true while landings stand.
The pusher then runs a no-op push and its guard's run-scope reconcile once per interval, with no end.
For a dropped landing this machine owns, D5 and D13 keep the pair until `lym fabric remove`.
Until then the `HeldRecords` entry keeps D3's owed period open, the all-clear never posts, D4's minutes keep counting, and the summary lists as owed a branch nothing can push.
Card 5's row "a level pass … keeping it while it holds a landing" pins a state the pass never reaches.
The guard-reach lens raised the unreachable branch at MEDIUM.
I raise it to BLOCKING for the summary, bar and all-clear effects across cards 5, 6, 20 and 21.

### [BLOCKING] The unrecorded-landing refusal names a dead way forward

**Location:** `_lym/plan/11-parent-pusher.md#Intent`

**Issue:** Step 4's way forward is `lym loom resume` in the run's task worktree.
`internal/loomcli/resume.go:88` answers a done run with "the run is done; there is nothing to resume".
The refusal is reached for runs past Finalize: after a lost ledger, after F3's removals, and after step 2 drops a landing with other local commits on it.
In the last case, step 2's rebase refusal and this refusal fall on the same branch's one record.
PATTERN-refusal-way-forward requires a way forward that works and a test that reaches it.

### [BLOCKING] Landing state lives only in a `.lym` cache

**Location:** `_lym/plan/11-parent-pusher.md#Intent`

**Issue:** PATTERN-lym-files says nothing a landing depends on lives only under `.lym`, and an absent cache reads as empty and refuses nothing.
The plan keeps the pending `done` write and the records hold only in the ledger under `<HubScratchDir>/push-ledger/`.
If the ledger is lost, the next pair push sends the held records commit.
Worktree-Teardown then reads both tips on origin and no entry, and removes the pair, so the board entry is never marked `done`.
Origin then reads the run's records while its parent lacks the work: the state D5 and the third Added-after-Discussion entry exist to prevent.
The `Lym-Landing` trailer card 12 adds gives the plan what it needs to rebuild the entries from git.

### [BLOCKING] New tests stand in for repository modules

**Location:** `_lym/plan/11-parent-pusher.md#Intent`

**Issue:** `ParentDeps` is filled in production by `loomcli#pusherDeps` from `fabricengine`, `boardengine` and `pairteardown`, so `TestParentPush`'s fakes simulate those modules, which PATTERN-test-stand-ins forbids for new tests.
The same holds for:
- card 13's `ParentPushState` rows;
- card 17's fake `Push` returning `ErrPushLockBusy` and its fake `Start`;
- card 19's fake `landingStands`, which `pushledger` fills.

The focus asked whether D5's and D8's criteria are reached through the real wiring (`landLocally`, the `ParentDeps` seams `pusherDeps` fills, `PushSummaryDeps`).
For `ParentDeps` and `catchUpPushSeams` they are reached only through fakes, and `landLocally` has no test at all.

### [BLOCKING] The new test package has no `tmuxkit.Main`

**Location:** `_lym/plan/04-push-ledger.md#Create`

**Issue:** `internal/pushledger` gets test files but no `TestMain`.
`TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain` (`cmd/lym/tmuxisolation_test.go:266`, tier0) fails it under PATTERN-test-isolation.

### [BLOCKING] `ArchiveParentMerge` becomes test-only

**Location:** `_lym/plan/12-finalize-local-landing.md#Delete`

**Issue:** The only production caller of `fabricengine.ArchiveParentMerge` is `loomcli#parentMergeArchiver` (`internal/loomcli/landingdeps.go:928`), which card 12 deletes.
The function then breaks PATTERN-test-only-code.
Its test, `internal/fabricengine/archiveparent_integration_test.go:18`, and its line in `internal/fabricengine/doc.go` stay.

### [MEDIUM] `lym beam pr` opens its pull request on a head origin lacks

**Location:** `_lym/plan/09-records-hold.md#Intent`

**Issue:** `PushBeam` goes through `PushRun`, which card 9 caps below a beam's landing commits.
`lym beam pr` then opens or edits the pull request with `pair.HeadSHA()` (`internal/beamcli/pr.go:81`, `:175`), the local head.
No card names this path or the beamcli doc lines that describe it.

### [MEDIUM] A spawn-owner test row breaks under card 7

**Location:** `_lym/plan/07-spawns-start-the-pusher.md#Edit`

**Issue:** `TestStartDetachedPush_RegistersTheChild` row `PairPushOwnsUnderItsSlug` (`internal/fabricengine/spawn_linux_test.go:38`) expects owner `task`.
Card 7 registers every spawn under the hub owner, so the row fails, and card 7 does not list the test.

### [MEDIUM] dobbyengine's doc goes stale

**Location:** `_lym/plan/03-driver-wait-readers.md#Uses`

**Issue:** Cards 3 and 20 change the run watch and add the push summary, but list `internal/dobbyengine/doc.go` only under Uses.

### [MEDIUM] pairteardown's doc goes stale

**Location:** `_lym/plan/19-teardown-hands-to-dobby.md#Uses`

**Issue:** Cards 9 and 19 change what `PushRun`, `HandOver` and the teardown composite do, but list `internal/pairteardown/doc.go` only under Uses.

### [MEDIUM] The landing's records commit can reach origin before its hold

**Location:** `_lym/plan/12-finalize-local-landing.md#Intent`

**Issue:** `LandLocally` commits the status records first and records the landing entry second.
A pair pusher already running from an earlier transition can push the new commit between the two, and a kill between them leaves it unheld.

### [MEDIUM] D1's spawn criterion is pinned for one verb of three

**Location:** `_lym/plan/07-spawns-start-the-pusher.md#Intent`

**Issue:** D1's second criterion names `lym board sync`, `lym fabric sync` and `lym fabric add`.
Card 7's tests reach only `Fabric.commit`, the path `lym fabric sync` takes, and `boardengine.spawnSync` alone.

### [LOW] PATTERN-shed-driver's background goes stale

**Location:** `_lym/plan/23-pattern-docs.md#Prosa`

**Issue:** `pattern/PATTERN-shed-driver.md:37` describes the capped re-step and hand-back that card 2 removes for network-class stops.

### [LOW] Cancel commits its intent before the landing refusal

**Location:** `_lym/plan/19-teardown-hands-to-dobby.md#Intent`

**Issue:** `lym batten cancel` writes and commits its intent (`internal/battencli/cancel.go:380-390`) before `EndForCancel`.
Card 15's past-the-merge read may make the cancel's verdict refuse earlier, so I keep the fork's LOW.
Where the `EndForCancel` refusal is reached, it follows a mutation.

### [LOW] Shared Decisions carry a misplaced `Applies to:` line

**Location:** `_lym/plan/00-overview.md:66`

**Issue:** The trailer decision carries two `Applies to:` lines, and the landing-entry decision above it carries none.
The second line lists the landing-entry decision's cards.

### [LOW] The hub clear time is unset before the first clear

**Location:** `_lym/plan/04-push-ledger.md#Intent`

**Issue:** `LastClear` reads zero until the last entry has gone once.
Until then the bar's minutes and the summary's period count from the zero time.

## Proposed-fix check

The fixes that add, drop or move targets were overlaid together through `lym loom validate-plan --overlay`:
F2, F8, F9, F10, F11, F12, F13 and F16, with the Uses removals they name from cards 5, 6, 8, 9, 10 and 15.
The overlaid set reported no blocking finding; the `{}` baseline reported none either.
A first combined run that left those Uses lines in place reported `uses-later-target` on cards 5, 6, 8, 9, 10 and 15, and `test-tier-unstated` on card 4.
A second run reported `card-field-empty` on card 8.
The fixes as written above carry the changes that cleared all three.
F17's added target, `internal/battencli#endRunPhase`, was overlaid alone afterwards and reported no blocking finding.
