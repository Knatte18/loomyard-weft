---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-fabric-prime-takeover.md:42
    summary: "Card 3 runs primeTakeovers only for a prime records push with an empty runSlug, but prime records pushes that name a slug (batten's kept cancel and re-adopt record pushes, pushOwedRunCommits, the detached coalesce push whose runSlug is its PauseSlug) still reach retryWeftPush's replay, which reads only that one run's takeover and replays a taken-over batten run's unpushed commits onto origin, breaking D6's never replayed."
    fix: "In card 3's retryWeftPush bullet, run primeTakeovers on every records worktree that is the prime's with a non-zero here, whatever runSlug is, and keep takeoverStop for a pair's records worktree alone. Add a TestRejectedRecordsPush_ReadsTheTakeover prime row that pushes with a run slug naming the run not taken over and asserts a *TakeoversError naming the taken-over run, with nothing pushed. In card 4, state that PushRun's *TakeoversError branch serves every prime push, a slug-naming one included."
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/10-teardown-pushes-first.md:36
    summary: "Card 10 adds Push and Level to battenshed.TeardownDeps as nil-legal seams, but the reflective seam tests over TeardownDeps then fail - envkit's nilLegal lists neither Teardown.Push nor Teardown.Level (TestFullEnv_FillsEverySeam) and loomcli's intentionallyNil lists only Teardown.Shutdown and Teardown.Remove (loom's TestWire_EverySeamFilled) - and card 10 lists those tests under Uses only, so it cannot pass its own gate."
    fix: "Add `internal/testkit/envkit#nilLegal` and `internal/loomcli#intentionallyNil` to card 10's **Edit:** list, and add an Intent line: nilLegal gains Teardown.Push and Teardown.Level, since nil is their documented default, and intentionallyNil gains both as batten-only seams battencli fills."
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/08-restarts-start-pusher.md:34
    summary: "Card 8 starts a pusher from DobbyRuns for a pair whose entry another machine owns, outside the code-tip settle's owned-elsewhere condition, relying on the pusher's reconcile to stop it; but skipsCheck skips the reconcile for a run at Finalize, so this machine pushes a run another machine owns (PATTERN-owner-machine: pushing a run only on its owner machine), and for every other row the guard posts its stopped-before-its-push notice on every dobby tick until the run is handed over."
    fix: "In card 8's DobbyRuns bullet, start the pair's pusher only inside the existing `!inputs.OwnedElsewhere && view.PairPresent` condition, beside the code-tip settle, and replace the bound with this one: a pair owned elsewhere gets no pusher from the tick, and the catch-up's HandOverOwnedElsewhere pushes that run's owed commits itself before it hands the run over."
  - id: F4
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/06-loom-pusher-verb.md:35
    summary: "Card 6's hidden `lym loom pusher` verb fails on a --worktree that does not resolve with the bare resolve error, a refusal reachable through a loom verb with no trailing way forward clause, which PATTERN-refusal-way-forward requires; the card's way forward, the next commit or dobby tick starting a pusher again, lives only in the Intent."
    fix: "In card 6's pusherCmd bullet, make the resolve failure's message end in `; way forward: run lym loom pusher --worktree with the path of an existing code worktree; the next commit, dobby tick or resume starts the pusher again`. Extend the planned TestResolvePersistentPreRun_SkipsSlugGroups row to assert that way-forward clause in the message."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/02-fabric-pusher-reads.md:42
    summary: "Card 10's push-then-Level loop never ends for a pair whose task branch origin deleted while its entry ended. The code side's checkOrigin skips the ended branch with a nil error, so Push returns nil and no ErrEntryEnded arises. TipsOnOrigin then reads the still-present, unpruned tracking ref as lacking the local code tip, so Level stays false and the teardown pushes again forever."
    fix: "In card 2's TipsOnOrigin paragraph, read a branch that origin no longer has while its entry is ended as held, as an absent tracking ref already reads, and add that row to TestTipsOnOrigin. In card 10's Call bullet, say that the Level read then lets the removal go on for such a pair."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-fabric-prime-takeover.md:79
    summary: "No planned row reaches the new PrimeRecords archive failure on its own. TestHandOverTips builds its request with Pair set, so a pre-receive hook refusing tags fails the pair's archiveOwnerSides first, before any prime archive runs. The planned ErrArchiveFailed, no mark assertion would therefore pass even with the prime-records archive missing. Card 4's archive-failure row also keeps the run's pair standing."
    fix: "In card 3's TestHandOverTips rows with PrimeRecords set, build the request with Pair nil (a batten run whose pair is gone here). Assert an owner archive tag naming the prime's local records tip before the mark, and, with the tag-refusing hook, an error wrapping ErrArchiveFailed and no hand-over mark."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/07-transitions-start-pusher.md:42
    summary: "Two docs that card 7 makes false are listed in no card. The `lym tender resume` help text in resumeCmd (internal/tendercli/cli.go:215) and the PushDeferred field doc of tenderengine.Deps (internal/tenderengine/report.go:114) both say that a run's transition pushes synchronously and owes no lift a push. Under the plan, a pause leaves a pair's commits unpushed until the dobby's tick restarts its pusher after the lift."
    fix: "Add `internal/tendercli#resumeCmd` and `internal/tenderengine#Deps` to card 7's **Edit:** list and `internal/tendercli/doc.go` and `internal/tenderengine/doc.go` to its **Uses:** list. Add an Intent line: both texts say that a lift spawns no push for a transition's commits, since the dobby's tick restarts the pair's pusher once no pause covers it."
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/01-fabric-owed-unrecorded.md:56
    summary: "Card 1 plans a new test, TestPushPairAnchored_OwedUnrecorded, whose rows copy the setups of existing rows in TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide. The transport row matches PrimeRecordsPushOwed_OriginLookupTransportFailure (refusedOriginURL), the teardown_lock row matches TeardownLockHeldPastTheWaitOwesTheRecordsPush, and the deferred row matches Deferred_HeldWeftWriteLock. The card already runs that test's BackoffSpent row in the mode, so the mode belongs in that table as a variant."
    fix: "Drop the `plan:internal/fabricengine_test#TestPushPairAnchored_OwedUnrecorded` Create target from card 1. Plan its rows as OwedUnrecorded variants of TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide's existing rows: the PrimeRecordsPushOwed_* rows, TeardownLockHeldPastTheWaitOwesTheRecordsPush and Deferred_HeldWeftWriteLock, each asserting the same class, IsPushOwed and no failed-push record. Add the not_fast_forward, ErrEntryEnded and commit-kept-in-order rows to the same test."
  - id: F9
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/08-restarts-start-pusher.md:52
    summary: "Card 8's new TestRestartPusher asserts only the warn-once-per-episode and recovery log lines of an episode set. TestEpisode_WarnsOncePerEpisodeAndInfoOnRecovery (internal/logger/episode_test.go:33) already pins that behaviour, and the log lines are human-read output that no code relies on."
    fix: "Drop the TestRestartPusher Create target and its Intent paragraph from card 8. RestartPusher's one contract, never failing its caller, is its void result, and the resume and dobby rows already exercise its calls."
  - id: F10
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/08-restarts-start-pusher.md:31
    summary: "Card 8 creates RestartPusher, a helper of runpusher's start shared by the battencli dobby read, beamcli's prime tick and both resume paths, in battencli and inside its consumers' card. A symbol reused across packages gets its own card. Its natural home is runpusher, whose start call the board entry names as the export the seams, the tick and the resumes share."
    fix: "Move the Create target to card 5 as `plan:internal/runpusher#RestartPusher` -> `func RestartPusher(start func() (bool, error), key string)`, with its Intent sentences. Remove `plan:internal/battencli#RestartPusher` from card 8 and list `plan:internal/runpusher#RestartPusher` under card 8's **Uses:**."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/01-fabric-owed-unrecorded.md:4
    summary: "The doc of fabricengine.StatusPushLockWait (internal/fabricengine/pushanchored.go:23) says commit-records and a run's transition push pass this wait. After cards 6 and 7, neither pushes; the pusher and Worktree-Teardown pass it, and no card lists the constant."
    fix: "Add `internal/fabricengine#StatusPushLockWait` to card 1's **Edit:** list, with an Intent line rewording its doc: the wait a worktree's pusher and Worktree-Teardown pass for the push lock and the pair's teardown lock."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-transitions-start-pusher.md:103
    summary: "Card 7's commit-records envelope reports pusher_started, but the StartPush it adds is a func() with no result, so the verb cannot know whether a pusher started, though runpusher.Start returns (bool, error)."
    fix: "In card 7, declare `StartPush func() bool` in statuscommit.Deps and in both commitStatusDeps types, returning runpusher.Start's started value with a failed start logged and read false. Have commitRecordsVerb put that value in pusher_started, and assert both values in TestCommitRecordsVerb_Success."
  - id: F13
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/02-fabric-pusher-reads.md:25
    summary: "RunCommitsAhead reads a records branch that origin no longer holds, whose entry ended and whose tracking ref is unpruned, as ahead. Its pusher's push then ends in the ErrEntryEnded error, which is neither owed nor refused. The pusher writes no record and exits, so every dobby tick until the teardown starts another pusher that fetches origin's board and exits."
    fix: "In card 2's RunCommitsAhead paragraph, read such a branch as level, as an absent tracking ref already reads, and add that row to TestRunCommitsAhead. Alternatively, in card 5, have Run's return on an error wrapping ErrEntryEnded leave a hand-over-like mark that Start reads; pick the first, which keeps the read local."
  - id: F14
    severity: NIT
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/08-restarts-start-pusher.md:25
    summary: "Card 8 makes the prime dobby's Tick restart the prime's pusher before its other work. internal/beamcli/doc.go's dobby-seam section says Deps.Tick runs after the tick's other steps and names the prime dobby's hub-wide duties without the pusher. Card 8 lists that doc under Uses, not Edit."
    fix: "Move `internal/beamcli/doc.go` from card 8's **Uses:** to its **Edit:** list. Add an Intent line: the dobby-seam section names the prime's pusher restart among the prime dobby's hub-wide duties, at the start of its Tick."
  - id: F15
    severity: NIT
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-fabric-prime-takeover.md:83
    summary: "The file header of internal/fabricengine/pushowed_test.go calls IsPushOwed the classification a transition seam retries a failed push by. After card 7 the pusher retries by it, and card 3 edits TestIsPushOwed in that file but leaves the header."
    fix: "In card 3's TestIsPushOwed sentence, add that the file header of pushowed_test.go names the pusher's owed retry instead of a transition seam's."
---

### [BLOCKING] A slug-naming prime push still replays a taken-over batten run

**Location:** `_lym/plan/03-fabric-prime-takeover.md:42`

**Issue:** D6 says a taken-over run's unpushed transitions "are never replayed".
Card 3 gates `primeTakeovers` on an empty `runSlug`.
Prime records pushes that pass a slug keep the old path at `pushclassified.go:827`:
`takeoverStop` reads that one run's entry, and `weftEndedByOrigin` sets aside only the runs origin ended.
Those pushes are batten's `Push` for the cancel record and the re-adopt record, which card 7 keeps (`commitstatus.go:172`, `cancelcmd.go:498`, `readopt.go:99`), `pushOwedRunCommits` (`pairteardown/handover.go:185`), and the coalesce push, whose `runSlug` falls back to `PauseSlug`.
With transitions no longer pushed, the prime commonly holds several batten runs' unpushed commits.
A cancel of run A after run B's takeover replays B's commits over B's takeover.

### [BLOCKING] Card 10's new TeardownDeps seams break the reflective seam tests it only lists

**Location:** `_lym/plan/10-teardown-pushes-first.md:36`

**Issue:** `envkit.go`'s full env fills `Teardown` with `Shutdown` and `Remove` only.
`nilLegal` (`internal/testkit/envkit/envkit.go:57`) lists neither new path.
`loomcli`'s `intentionallyNil` (`internal/loomcli/wiring_seams_test.go:14`, Teardown entries at :96-97) lists only `Teardown.Shutdown` and `Teardown.Remove`.
Card 10 names `TestFullEnv_FillsEverySeam`, `TestNilSeams_*` and `TestWire_EverySeamFilled` under `Uses` but edits neither map, so its own gate fails.

### [BLOCKING] The dobby starts pushers for pairs another machine owns

**Location:** `_lym/plan/08-restarts-start-pusher.md:34`

**Issue:** Card 8 places the pair's `StartIfAhead` "outside its owned-elsewhere condition".
Its stated bound is that the pusher's reconcile stops such a pair (card 6).
That bound fails at Finalize: `skipsCheck` (`internal/loomcli/prepush.go:91`) returns skip for a run at Finalize, so the guard lets the push through.
D7 keeps that skip because Finalize judges its owners itself, but that judgment runs only when Finalize runs on the owner machine, not in this dobby's pusher.
`PATTERN-owner-machine`, by its declared text, allows pushing a run only on its owner machine.
For every other row the guard's owned-elsewhere stop posts a notice on each start (`prepush.go:156-158`), so each tick posts one more notice for as long as the pair stays ahead.
D5's "every pair of its runs" does not require pushing a run owned elsewhere.
The catch-up already pushes such a run's owed commits through `pushOwedRunCommits` before `HandOverOwnedElsewhere` hands it over.

### [BLOCKING] The pusher verb's resolve refusal names no way forward

**Location:** `_lym/plan/06-loom-pusher-verb.md:35`

**Issue:** `PATTERN-refusal-way-forward` binds `internal/loom*`.
Every refusal a bound verb can return ends in a `way forward:` clause, and `refusal-spec.md`'s transient class says "the message names the retry".
Card 6 fails the verb "with the resolve error".
The retry ("the next commit or dobby tick starts a pusher again") is written only in the Intent, not in the message, and the planned row asserts only the resolve error.

### [MEDIUM] Worktree-Teardown loops forever on an ended, deleted task branch

**Location:** `_lym/plan/02-fabric-pusher-reads.md:42`

**Issue:** Card 1 wraps `ErrEntryEnded` around the records side alone (`recordsSideNotLanded`).
A code side whose branch origin deleted while its entry ended takes `checkOrigin`'s `view.ended` path (`pushclassified.go:641-642`) and returns nil, so card 10's `Push` succeeds.
`TipsOnOrigin` treats only an absent tracking ref as held.
An unpruned tracking ref that lacks the local tip reads not held, so card 10's `Level`-false branch pushes again without end.

### [MEDIUM] No row isolates the new prime-records archive failure

**Location:** `_lym/plan/03-fabric-prime-takeover.md:79`

**Issue:** `TestHandOverTips`'s request sets `Pair: d.location` (`handover_integration_test.go:54-55`).
`HandOverTips` archives the pair's sides first (`internal/fabricengine/handover.go:88-91`), so a tag-refusing hook fails there.
The planned assertion of `ErrArchiveFailed` with no mark then holds whether or not the `PrimeRecords` archive exists or runs before the mark.
Card 4's archive-failure row stands its pair too, so neither card reaches the branch.

### [MEDIUM] The tender resume help and PushDeferred doc still state the synchronous transition push

**Location:** `_lym/plan/07-transitions-start-pusher.md:42`

**Issue:** `internal/tendercli/cli.go:215-217` (in `resumeCmd`) tells the user "A run's transition pushes synchronously, so no pair push is spawned for one."
`internal/tenderengine/report.go:114` (on `Deps.PushDeferred`) says the same.
Card 7 makes both false, and no card lists either.
Card 11 cannot carry them, since they are Go code, not Prosa targets.
The fix was checked with `lym loom validate-plan --overlay`, which needs the two packages' `doc.go` under `Uses`.

### [MEDIUM] Card 1's new test function duplicates the existing owed-push rows

**Location:** `_lym/plan/01-fabric-owed-unrecorded.md:56`

**Issue:** `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` already builds every owed setup card 1's new test lists (`pushanchored_integration_test.go:395-425`, `575-620`, `796-826`):
`PrimeRecordsPushOwed_OriginLookupTransportFailure` sets `refusedOriginURL`, `TeardownLockHeldPastTheWaitOwesTheRecordsPush` holds the teardown lock, and `Deferred_HeldWeftWriteLock` holds the weft write lock.
The new behaviour is the mode, which is one more dimension of those rows.
Card 1 already treats it so for `BackoffSpent`, and the testing rules make a new case a row in the test that covers the behaviour.

### [MEDIUM] TestRestartPusher re-tests the episode set's logging

**Location:** `_lym/plan/08-restarts-start-pusher.md:52`

**Issue:** The planned rows (a success; a failing start logs one Warn over two calls; a success after it logs the recovery) are the episode-set contract.
`TestEpisode_WarnsOncePerEpisodeAndInfoOnRecovery` in `internal/logger` already pins that contract.
They assert log lines only a human reads.
No other behaviour of `RestartPusher` is left to pin.

### [LOW] RestartPusher sits in battencli inside its consumers' card

**Location:** `_lym/plan/08-restarts-start-pusher.md:31`

**Issue:** `RestartPusher` is exported from `battencli` and called from `beamcli`'s `hubDobbyDeps` and from the resume seams.
It wraps `runpusher`'s start and nothing of batten's.
The granularity rule gives a cross-package symbol its own card.
Card 5, the `runpusher` card, is that card, and the move keeps every restart path's helper beside `Start`.
The moved target was checked with `lym loom validate-plan --overlay`.

### [LOW] StatusPushLockWait's doc names pushes that no longer use it

**Location:** `_lym/plan/01-fabric-owed-unrecorded.md:4`

**Issue:** The constant's doc (`internal/fabricengine/pushanchored.go:23`) names "`commit-records` and a run's transition push", which cards 6 and 7 replace with the pusher's push.
Card 1 already covers `fabricengine/doc.go`, so the gate admits the target there.

### [LOW] pusher_started has no value to report

**Location:** `_lym/plan/07-transitions-start-pusher.md:103`

**Issue:** `StartPush func()` carries no result, and card 7 says it "logs its own failure".
The envelope key card 7 adds therefore has no source, unless the field returns what `runpusher.Start` reports.

### [LOW] An ended records branch with a stale tracking ref restarts a pusher every tick

**Location:** `_lym/plan/02-fabric-pusher-reads.md:25`

**Issue:** Card 2 reads only an absent tracking ref as level.
A branch whose origin copy is gone, while its tracking ref is still present, reads ahead.
The pusher's push ends in the `ErrEntryEnded` error (`pushanchored.go:358`), and card 5's `Run` returns on it, since it is not owed.
Nothing stops the next tick's `StartIfAhead` from repeating the fetch and exit until the teardown.

### [NIT] beamcli/doc.go's dobby-seam section omits the pusher restart

**Location:** `_lym/plan/08-restarts-start-pusher.md:25`

**Issue:** Card 8 adds the prime's pusher restart to the prime dobby's `Tick`, which `internal/beamcli/doc.go` documents, but it lists that doc as unchanged.

### [NIT] pushowed_test.go's header still names the transition seam

**Location:** `_lym/plan/03-fabric-prime-takeover.md:83`

**Issue:** Card 7 removes the transition seam's owed retry, but the file header of `internal/fabricengine/pushowed_test.go` still names it as the reader of `IsPushOwed`.

## Proposed fixes checked against the plan gate

The fixes that add, drop or move targets (F2, F7, F8, F9, F10, F11 and F14) were overlaid together with `lym loom validate-plan --overlay`.
The `{}` baseline reported no blocking finding.
The combined run also reported none, once F7 listed the two tender `doc.go` files under `Uses` and F11 moved `StatusPushLockWait` to card 1, which already covers `fabricengine/doc.go`.

## Focus verdicts

- Fidelity: D6's prime archive is carried by cards 3 and 4.
  D5's dobby ahead-of-origin read is carried by cards 2 and 8.
  D7's pause-before-Finalize order is carried by card 5's `Run` and card 6's `Guard`.
  D8's unset `PushOwed` is carried by card 1 and the existing `runread.go` read.
  Every Decision and Constraint has a carrying card; F1 is a hole in how card 3 carries D6.
- Card order: no card uses a symbol a later card creates.
  `StartPush` is a field card 7 adds itself, and `runpusher.Start` comes from card 5.
- Refusal reach in cards 3 to 6: reachable, except the prime-records archive failure (F6).
- Attack surface: card 6's verb, card 1's mode and card 7's removal of the transition-time reconcile each state their bound.
  `skipsCheck` is not changed, by the plan's shared decision, which D7 admits since the pusher reads the pause first.
  Card 8's bound for owned-elsewhere pairs does not hold (F3).
- Edit typing: every Edit card names member glyphs.
  No signature change lacks the re-sign arrow.
  No `Extends` target rewrites existing content.
- Research steps, board scope, filed tasks: none.
  The "under `go test` the spawn starts nothing" fact holds through `selfexec.LymBinary`'s `ErrTestBinary`.
- Verify lines: no card carries one.
  Card 11's `PATTERN.md` lines match the declared texts.
  No card breaks `PATTERN-fabric-boundary`, and `PATTERN-daemon-wakeups` holds for the pusher's backoff and re-reads.

## Rejected

- D7's "an entry owned elsewhere is handed over" read as a pusher-side hand-over: card 6's reading, that the catch-up hands it over, matches `PATTERN-owner-machine`'s own text, "An entry owned elsewhere is handed over by the catch-up".
- Card 5's `TestStart` and `TestRun`: a new package with no existing test of either behaviour.
- Card 2's three read tests: three distinct public reads, and no existing test covers any of them.
- Card 8's `TestBattenPreRun_ResumeStartsThePusher`: the pusher start on a resume is new behaviour, which `TestBattenPreRun_ReopensTheWindowWithNoClientAttached` does not pin.
- Integration tests that assert origin after a run: `TestBattenIntegration_Rows`' remote assertions survive card 10, since its push lands before the removal, and `TestAdoptIntegration_SecondCloneTakesOver` pushes its fixture explicitly.
