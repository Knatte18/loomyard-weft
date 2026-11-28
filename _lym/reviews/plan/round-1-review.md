---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/07-finalize-queue-entry.md:66
    summary: "Cards 7 and 8 nest new structs in `landingshed.Deps` (`landingqueue.BackstopSeams`, `DetachDeps`) but leave `envkit`'s `TestNilSeams_ReportsExactPaths` and `TestNilSeams_SkipsNilLegalPaths` under Uses with no new marker, so the tier0 seam scan refuses both tests."
    fix: "In card 7, move `internal/testkit/envkit#TestNilSeams_ReportsExactPaths` and `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths` from **Uses:** to **Edit:**, and add both to the Intent's list of tests that gain `//lym:seam internal/landingqueue#BackstopSeams`. In card 8, make the same move and add both to the list that gains `//lym:seam internal/landingshed#DetachDeps`. In both cards, replace the search `grep -rln \"lym:seam internal/landingshed#Deps\" internal cmd` with one that lists each marked test function, not each file, since envkit_test.go holds three marked tests (lines 36, 115 and 167). The combined `validate-plan --overlay` run with this move passes."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-finalize-batch-leader.md:106
    summary: "Card 8 leaves the leader's own chain link undefined when `BuildLandingChain` returns it Conflicted or Held. That is reachable after `syncParent` fast-forwards the parent under the lock, and card 9's k = 0 culprit rule assumes chain commit 1 is the leader's."
    fix: "In card 8's `Taking the batch` paragraph, after `syncParent` passed and before `BuildChain`, have the leader ask card 7's `mergeNeedsSession`. One that would need a conflict session frees the lock, runs `mergeUnlocked` and takes the lock again at the front, building no chain (D8's last bullet). A leader whose synced parent now holds its work (`parentHoldsWork` read again after the sync) takes card 7's `landHeld` path and leads no batch. In `Landing the prefix`, state that a Conflicted or Held link of the leader's own, as after a parent moved between that check and the build, sets no outcome on the leader's entry. That leader builds no chain and frees the lock as for a needed session. Add one sentence to card 9 that the k = 0 culprit is the leader because chain commit 1 is always the leader's link. Add two rows to `TestFinalizeQueue_LandsABatch`. In the first, a parent fast-forwarded under the lock to an upstream commit that conflicts with the leader builds no chain, frees the lock before any merge and takes no member's entry. In the second, a parent fast-forwarded to an upstream commit holding the leader's work takes the held path and leaves the waiting entries with no outcome."
  - id: F3
    severity: BLOCKING
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/02-gitrepo-commit-tree.md:16
    summary: "Card 2's `WriteCommitTree` calls `runChecked`, so it must join `cmd/lym`'s `gitrepoPinnedRunBoundMethods`, which no card edits. The card also says the message is read from standard input, which `runChecked` cannot pass (gitrepo.go:83)."
    fix: "Add `cmd/lym#gitrepoPinnedRunBoundMethods` (cmd/lym/gitrepoboundary_test.go, package main) under card 2's **Edit:**, with `WriteCommitTree` added to the list. Replace `message read from standard input` with `-m <message>` passed through `Repo.runChecked`, which keeps the call on the one pinned route. The combined `validate-plan --overlay` run with this target passes."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/07-finalize-queue-entry.md:75
    summary: "D6 has every taker of the parent lock run the backstop first. `parentMergeTip` (parentalign.go:277) takes the parent lock through `guard.ensure` for Parent-Align and for card 5's lock-free merge, yet no card runs the backstop there, and the Shared Decision `fabric-lock-takers` names only fabriccli."
    fix: "Add `internal/landingshed#parentMergeTip` under card 7's **Edit:**. Under the lock, right after `guard.ensure`, it runs `deps.Queue.Backstop(deps.Backstop)` when `deps.Queue` is set, prunes nothing and returns a backstop error as its error. Say so in the Intent beside the `landUnderLock` sentence. Add a row to `TestFinalize_QueueEntryAndOutcome`: a lock-free merge whose tip read takes the parent lock while a move mark stands calls `WriteLanding` for the mark's held commits before it reads the tip. The combined `validate-plan --overlay` run with this target passes."
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/08-finalize-batch-leader.md:101
    summary: "Card 8's restore-failure Stuck has a way forward, clean and `lym loom resume`, that works only through card 10's shed restore. Card 10 comes after card 8, which breaks the plan spec's rule that no card depends on a later one."
    fix: "Move card 10 (`resume-restores-detached`) ahead of card 8 with `lym loom remove-card` and `lym loom insert-card`. It uses only card 4's `RestoreDetached` and `ReadDetachState` and card 7's `queueHub`, so it needs nothing from cards 8 or 9. Then reword card 8's `which card 10's shed restore then completes` and card 4's `(cards 8 and 10 word those stops)` to the new card numbers. Card 10's `TestFinalizeQueue_ResumedDetached` builds its chain commit with `BuildLandingChain` from card 3 rather than through a leader."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-finalize-queue-entry.md:101
    summary: "Card 7 says a run whose `PullRequest.Mode` read fails writes no entry and lands alone. Under the lock, `pullRequestLanding` halts on the same failed read (finalize_pr.go:266-268), so the run halts rather than lands, and the row cannot pass."
    fix: "In card 7's Join paragraph, replace `or fails to read (D10: that run writes no entry and lands alone through today's path)` with `or fails to read, which writes no entry, and the decision under the lock then halts on the same failed read`. Change the row to: a run whose `PullRequest.Mode` read fails writes no entry and halts under the lock with `LandingStopPrefix`."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/07-finalize-queue-entry.md:65
    summary: "Cards 7 and 11 define `UpstreamLacks` as the negation of `IsAncestor` against origin's tracking ref. `IsAncestor` errors on an absent ref (ancestry.go:28-37), and card 1 keeps the mark on a seam error, so a parent with no tracking ref wedges every later Finalize, `lym fabric anchor` and `lym fabric pull --hub`."
    fix: "In card 7's `landingDeps` sentence, state that `UpstreamLacks` reads an absent `refs/remotes/origin/<parent>` as lacking the commit, as `localBranch.lacks` reads an unset upstream (pusherreads.go:79-88). It must not return the ref read's error. State the same for card 11's `queueBackstop` against the branch's upstream. Add a row to card 11's `TestAcquireBranchLock_WritesTheLandingsAStandingMarkOwes`: a parent with no tracking ref writes the mark's landings and removes the mark."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-finalize-batch-leader.md:115
    summary: "Card 8's leader writes one ledger landing per prefix entry, and the backstop's `WriteLanding` writes too, with no `Deps.PushSkipped` check. Today's `pushParent` records no landing under the push skip (finalize.go:1045-1053)."
    fix: "In card 8's move paragraph, state that under `Deps.PushSkipped` the leader writes and removes the move mark and sets each entry `landed`, but calls no `WriteLedgerLanding`. Each member's own Finalize then takes the held path, whose `pushParent` commits its records and marks its task done as today. In card 7, state that `landingDeps` fills `Backstop.WriteLanding` as a no-op under `pushSkipped`. Add a row to `TestFinalizeQueue_LandsABatch`: with pushes skipped, a batch of two lands with no ledger landing written and each member ends Done."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/10-resume-restores-detached.md:44
    summary: "Card 10 wraps a resume-restore refusal only in the new `ErrRestoreRefused`. The step driver maps only `ErrLockedCheckRefused` onto its refusal kind (shedverbs step.go:410, spec.go:211), so a refused restore reaches the loop as an unclassified error."
    fix: "In card 10, have `restoreUnderRunLock` wrap a restore error in both sentinels, as `fmt.Errorf(\"%w: %w: %w\", ErrLockedCheckRefused, ErrRestoreRefused, err)`. The step driver's existing `errors.Is(err, shedengine.ErrLockedCheckRefused)` branch then reports it as a drive refusal with its trailing way forward. Have the `TestLockedCheck_RefusesUnderTheRunLock` rows assert that `errors.Is` holds for both sentinels."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-finalize-queue-entry.md:107
    summary: "Card 7's row says a backstop `WriteLanding` error runs no merge. The card's own `Call` order runs `mergeUnlocked`'s lock-free merge before `landUnderLock` reaches the backstop, so the row asserts something false."
    fix: "Reword the row: a backstop `WriteLanding` error ends `Call` with that error, runs no merge under the lock and makes no parent-side merge."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/04-fabric-detached-checkout.md:45
    summary: "Card 4 says `CheckoutDetached` and `CheckoutBranch` run `git checkout` through `gitrepo`, but no gitrepo method runs a checkout and no card creates one."
    fix: "Say that both run `git checkout` through `gitexec.Run` in the code checkout, as fabric's other plumbing does (leftovers.go:71). Do not route them through gitrepo, which would need new methods and entries in `cmd/lym#gitrepoPinnedRunBoundMethods`."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/04-fabric-detached-checkout.md:27
    summary: "fabricengine's package doc (doc.go:141) says `RunCommitsAhead`, `RefusedPushStands` and `TipsOnOrigin` read local refs and failed-push records only. Card 4 makes them read the detach marker too, and lists the doc only under Uses."
    fix: "Move `internal/fabricengine/doc.go` from card 4's **Uses:** to its **Edit:**. Have the Intent reword that sentence to say the reads take local refs, failed-push records and the worktree's detach marker. The combined `validate-plan --overlay` run with this move passes."
  - id: F13
    severity: LOW
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/08-finalize-batch-leader.md:146
    summary: "Cards 4, 8 and 10 create test functions with no file named. Card 4's `TestPushPairAnchored_HoldsOffTheCodeSideOfADetachedPair` names no tier either. Cards 8 and 10 put tier1 tests in loomcli, whose `landingdeps_test.go` is tier0."
    fix: "In card 4, place `TestPushPairAnchored_HoldsOffTheCodeSideOfADetachedPair` in the tier1 `detach_integration_test.go` the card creates. In cards 8 and 10, place `TestLandingDeps_TaskHeadReadsTheBranchWhileDetached` and `TestNewShed_RestoresADetachedTaskWorktree` in loomcli's tier1 `landlocally_integration_test.go` (tiergate.Require(t, 1) at line 105), or in a new `_integration_test.go`. Name the file in each card's prose."
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/10-resume-restores-detached.md:60
    summary: "Two acceptance criteria name Tier 1 in `internal/landingshed`, but their tests sit in `internal/loomcli`. One is D5's `resumed at any row ... put back on its task branch first`, which card 10 plans only in `TestNewShed_RestoresADetachedTaskWorktree`. The other is D6's `a member's pair pusher sends nothing past the records commit`, which card 8 plans in `TestWriteLedgerLanding_AChainOfOwnTipsReadsAsOwed`."
    fix: "In card 10 and in card 8, add one sentence each saying that the criterion is planned in `internal/loomcli` because the resume restore and the ledger hold are wired there, and `queueHub` in `landingshed_test` cannot build loomcli's shed or ledger writer. The code review then reads the criterion as met at that package."
  - id: F15
    severity: LOW
    class: scope
    origin: handler
    location: _lym/plan/03-fabric-landing-chain.md:42
    summary: "Card 3 caps `NewestLandingOf` at `maxLandingWalk`, 500 first-parent commits, and reports none past it. D8 asks for the newest commit on the whole first-parent history whose trailer names the slug, and the decision record licenses no bound."
    fix: "Drop `maxLandingWalk` and the 501-commit row from card 3. Have `NewestLandingOf` walk the whole first-parent history in one `git log --first-parent` read, as `commitsPastUpstream` lists a range today (landing.go). A bound needs a decision-record entry first, and only the parent can record one."
---

### [BLOCKING] The envkit nil-seam tests miss the markers for the new nested Deps structs

**Location:** `_lym/plan/07-finalize-queue-entry.md:66`, and card 8's matching sentence at `_lym/plan/08-finalize-batch-leader.md:88`

**Issue:**
- Card 7 adds `Deps.Backstop landingqueue.BackstopSeams`, and card 8 adds `Deps.Detach DetachDeps`.
- Both of envkit's `TestNilSeams_*` tests carry `//lym:seam internal/landingshed#Deps` and call `NilSeams(env)` (envkit_test.go:115-131, 167-193).
- `pattern/PATTERN-test-speed.md:174-181` has the tier0 scan refuse a test that reaches `envkit.NilSeams` while its markers miss a struct the `Env` set holds.
- The cards list both tests under Uses, and `seam-test-unlisted` passes a test listed under Uses that the card breaks (loom-plan-checks-spec.md:271), so no gate catches it.
- The method fault: the cards' search `grep -rln` lists files, and envkit_test.go holds three marked tests.

### [BLOCKING] The leader's own chain link may come back Conflicted or Held, and no card says what happens

**Location:** `_lym/plan/08-finalize-batch-leader.md:106`

**Issue:**
- Card 8 runs `syncParent` after `actOnOutcome` answered none.
- `syncParent` can fast-forward the parent to an upstream tip (finalize.go:755-766). That tip may conflict with the leader's task head, or hold its work through a remote squash.
- `BuildLandingChain` then returns the leader's own link Conflicted or Held. Card 8 defines those outcomes for members only.
- Card 7's `mergeNeedsSession` precheck guards merge-ins under the lock, not the chain build.
- Card 9's k = 0 rule, where the leader is the culprit, holds only if chain commit 1 is the leader's link.

### [BLOCKING] WriteCommitTree breaks the gitrepo pinned-method list, and its stdin claim does not fit runChecked

**Location:** `_lym/plan/02-gitrepo-commit-tree.md:16`

**Issue:**
- `cmd/lym/gitrepoboundary_test.go` pins `gitrepoPinnedRunBoundMethods`, the gitrepo methods whose bodies call `r.run` or `r.runChecked`.
- A new method that calls `runChecked` must join that list, and no card targets it.
- `Repo.runChecked` is `gitexec.Run(args, r.path)` (gitrepo.go:83), which passes no stdin, so the card's stdin message cannot go through it.

### [MEDIUM] parentMergeTip takes the parent lock without the backstop

**Location:** `_lym/plan/07-finalize-queue-entry.md:75`

**Issue:**
- D6's backstop binds every taker of the parent lock.
- The repository's `parentlock.Store.Acquire` callers are `parentguard.go:78` and `fabriccli/anchordeps.go:567`.
- The guard is entered from `landUnderLock` and from `parentMergeTip`. The latter runs for Parent-Align (parentalign.go:147) and for the lock-free merge card 5 keeps (finalize.go:387), and it holds the lock to read the local tip (parentalign.go:277-281).
- Card 7 runs the backstop only in `landUnderLock`, and card 11 only in `acquireBranchLock`.

### [MEDIUM] Card 8's restore-failure way forward depends on the later card 10

**Location:** `_lym/plan/08-finalize-batch-leader.md:101`

**Issue:**
- Card 8 ends a failed restore Stuck with the way forward to clean the worktree and run `lym loom resume`, `which card 10's shed restore then completes`.
- Until card 10 lands, that resume runs Finalize over a detached worktree with no restore.
- The plan spec's `Plan vs. schedule` section requires that a card never depends on anything a later card does.

### [MEDIUM] A failed PullRequest.Mode read halts under the lock, not lands alone

**Location:** `_lym/plan/07-finalize-queue-entry.md:101`

**Issue:**
- `pullRequestLanding` reads `pr.Mode()` under the lock and halts on its error with `LandingStopPrefix` (finalize_pr.go:266-268).
- finalize.go:385 and :431 say the decision under the lock ends the call on a failed read.
- So card 7's row and its D10 parenthesis assert a landing that cannot happen.

### [MEDIUM] The backstop's UpstreamLacks errors on a parent with no tracking ref

**Location:** `_lym/plan/07-finalize-queue-entry.md:65`, and card 11 at `_lym/plan/11-fabric-lock-backstop.md:23`

**Issue:**
- `gitrepo.IsAncestor` returns an error for every exit but 1 (ancestry.go:28-37), so a missing `refs/remotes/origin/<parent>` is an error.
- Card 1's `Backstop` keeps the mark on a seam error, so every later lock taker fails again.
- That state is reachable: `ParentNoUpstream` is a legal relation (finalize.go:740), and a leader can die after the move.
- Card 11's way forward, clear the cause, then names nothing the operator can clear.

### [MEDIUM] Ledger landings are written under the push skip

**Location:** `_lym/plan/08-finalize-batch-leader.md:115`

**Issue:**
- `pushParent` records no landing under `Deps.PushSkipped` and marks the task done itself (finalize.go:1045-1053).
- Card 8's `WriteLedgerLanding` loop and the backstop's `WriteLanding` carry no such check.
- So a hub with pushes off gets ledger landings that its own single-run path never writes.

### [MEDIUM] The step driver does not classify ErrRestoreRefused

**Location:** `_lym/plan/10-resume-restores-detached.md:44`

**Issue:**
- shedverbs step.go:410 maps `errors.Is(err, shedengine.ErrLockedCheckRefused)` onto `StepBusyKind`, and spec.go:211 documents that mapping.
- A restore refusal wrapped only in the new `ErrRestoreRefused` falls through to the generic error path. Card 10 does not list shedverbs.

### [LOW] Card 7's backstop-error row claims no merge ran

**Location:** `_lym/plan/07-finalize-queue-entry.md:107`

**Issue:**
- Card 7's own `Call` order runs `mergeUnlocked`, then `joinQueue`, then `landUnderLock`.
- The backstop runs inside `landUnderLock`, so the lock-free merge has already run when it fails.

### [LOW] Card 4's checkouts name a gitrepo route that does not exist

**Location:** `_lym/plan/04-fabric-detached-checkout.md:45`

**Issue:**
- internal/gitrepo has `MergeFFOnly` (merge.go:266) but no checkout method, and no card creates one.

### [LOW] fabricengine's package doc goes false under card 4

**Location:** `_lym/plan/04-fabric-detached-checkout.md:27`

**Issue:**
- `internal/fabricengine/doc.go:141` says the three pusher reads take local refs and failed-push records only.
- Card 4 makes them read the detach marker file too.

### [LOW] New test functions name no file, and loomcli's tier1 tests would land in a tier0 file

**Location:** `_lym/plan/08-finalize-batch-leader.md:146`

**Issue:**
- `PATTERN-test-speed` gives each test file one tier.
- The three named functions have no file, and loomcli's `landingdeps_test.go` is tier0 (line 49).

### [LOW] Two acceptance criteria are planned outside the package they name

**Location:** `_lym/plan/10-resume-restores-detached.md:60`

**Issue:**
- The D5 resume-at-any-row criterion and the D6 pusher-hold criterion each name Tier 1 in `internal/landingshed`.
- Their only planned tests are in `internal/loomcli`.

### [LOW] NewestLandingOf's 500-commit cap is not in the decision record

**Location:** `_lym/plan/03-fabric-landing-chain.md:42`

**Issue:**
- D8 defines a held run's landing commit as the newest one on the parent's first-parent history whose trailer names it.
- The cap changes that answer for an old landing to the parent-head fallback, with no decision recording the bound.

## Rejected

- repo-invariants F5 (lymdirs/doc.go misses the new names): the doc lists only some constants and says each constant's own doc comment says where it sits (lymdirs/doc.go:8, 15), so the new names leave it true.
- repo-invariants F7 (shedengine/doc.go incomplete): doc.go:27-28 describes `LockedCheck` correctly and claims no other hook is absent, so card 10 makes no sentence false.
- Card 3's `MoveCodeTo` tip-not-descending branch, which guard-reach found unreachable from `landPrefix`: the lens itself read it as a fail-loud guard with a pinned row, not a defect.
