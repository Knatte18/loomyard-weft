---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/10-batten-resume-readopt.md (Tests, TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal rows)
    summary: Card 10 puts taskWorktreeGone rows that spawn git into a tier0 test, breaking PATTERN-test-speed's rule that tier0 spawns nothing
    fix: In card 10's Tests, keep only the row "no path reads gone" in TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal (tier0). Move the rows "a plain file and an empty directory at the path read gone" and "a resolving worktree reads not gone" into TestBattenIntegration_GoneTaskWorktree (tier1, hubforge hub) as sub-rows calling taskWorktreeGone directly. Every row other than the absent path reaches lymcwd.ResolveWorktree, which runs git, and the resolving row needs a real worktree.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-add-adopt-only.md (Intent, refuseAdoptLocalCheckedOut bullet)
    summary: The records-branch-checked-out refusal of D2's closed set runs only when the warp branch is adopted locally, so an adopt-only Add over an origin task branch meets a plain worktree-add failure that card 12 treats as transient on every pass
    fix: "In card 4's Intent, add a bullet: under AdoptOnly, Topology.add calls refuseAdoptLocalCheckedOut whenever opts.AdoptLocal is set and the weft branch exists locally, not only when warpAdopted (add.go:341), so a records branch checked out in another records worktree returns *ErrAdoptLocalCheckedOut with Records true even when the task branch comes from origin. Add a TestAdd_AdoptOnly row: task branch only on origin, local weft branch checked out in another records worktree, errors.As *ErrAdoptLocalCheckedOut with Records true, requireNothingCreated."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-add-adopt-only.md (Intent, reused records worktree bullet)
    summary: When Add reuses a standing records worktree whose weft branch is behind origin, its fast-forward fetch into refs/heads/<weft> is refused by git because the branch is checked out there, a transient failure on every pass
    fix: "In card 4's reused-records-worktree bullet, add: when weftOld.fastForwardTo is set and the records worktree is reused, Add advances the branch with `git merge --ff-only <origin tip>` run inside the reused worktree (after fetching the tip by SHA through FetchWithArgs), instead of the fetch into refs/heads/<weft> at add.go:468, which git refuses for a branch checked out in a worktree. Add a TestAdd_AdoptOnly row: records worktree standing on a weft branch one commit behind origin, task worktree deleted, the pair is re-made and the weft branch sits at origin's tip."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-remove-archive-order.md (Intent, archiveWeftTip steps 1-3)
    summary: Step 2 compares the local tip with origin's tip before step 3 fetches origin's tip, so the ancestry test against a commit the records repository lacks errors and fails the removal in the diverged and ahead rows
    fix: "In card 6's step 1, add: when origin's tip differs from the local tip, fetch it by SHA (FetchWithArgs(origin, tip)) right after reading it, before any comparison. Step 2 and step 3 then compare commits both present locally, and step 3 no longer fetches."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-cleanup-teardown-lock.md (Intent, both sweeps)
    summary: Both sweeps take the per-slug try lock after they enumerated the live warp branches, the checked-out map and the tips, so an Add of the slug that finished in between is not seen and the remote warp sweep can delete a task branch a re-adopted pair just took over
    fix: "In card 8's Intent, add: once a slug's try lock is held, Topology.Cleanup re-reads under it whether a warp worktree now sits on the orphan's paired warp branch (List over the warp repo) and skips the slug as live when one does; Topology.cleanupRemoteWarp re-reads, under the lock, whether a hub worktree has the candidate checked out and keeps it with a Reason naming the live pair when one does. Add one row to TestCleanupRemoteWarp_ApplyDeletesOnlyTheCandidate that makes a worktree on the candidate's branch through the afterEnumerate hook and asserts the candidate kept and its origin branch standing."
  - id: F6
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/13-remove-follow-up.md (Wiring), internal/battencli/resumereconcile.go:127, internal/fabricengine/originreconcile.go:333-336
    summary: Batten's own resume refusal for a stopped entry surfaces fabric's gone-branch way forward naming lym fabric remove, the board entry's own case, and no card appends D3's follow-up there although batten can word it without touching fabric
    fix: "Add `internal/battencli#battenCLI.preflightRunOwner` to card 13's **Edit:** list. In its Intent, add a Wiring bullet: preflightRunOwner, which has already read the run's status, appends pairteardown.BattenFollowUp(slug, status, true, ownerHeld) to an ownerPreflight error whose text names `lym fabric remove`, with ownerHeld read as Teardown's ownerHeld seam reads it, so a resume refusal of a stopped entry that names the removal also names the cancel before Land or the Land way forward. Add a row to TestOwnerPreflight_GoneTaskWorktree or the existing ownerPreflight table: a stopped verdict whose branch way forward names `lym fabric remove` at Run-Shed yields an error naming `lym batten cancel <slug> --remove`. Checked with validate-plan --overlay; placing the edit on reconcileResume instead is refused by uses-later-target, since card 10 uses it."
  - id: F7
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/01-gitrepo-lease-and-retry.md (Intent item 1)
    summary: Card 1 makes "missing necessary objects" retryable for every PushWithRetry caller, but D4 step 4 licenses the retry only for the archive-tag push, and the widening changes PushRebaseFree's ErrPushRejected classification and Push's rebase recovery
    fix: "In card 1, add `plan:internal/gitrepo#PushWithRetryReasons` -> `func PushWithRetryReasons(push func() error, ref string, delays []time.Duration, sleep func(time.Duration), extraReasons []string) error` under a new **Create:** block, add `internal/gitrepo#PushWithRetry` under **Edit:**, and give `internal/gitrepo#retryablePushRefusal` the arrow -> `func retryablePushRefusal(err error, ref string, extraReasons []string) (line string, retryable bool)`. Rewrite Intent item 1 so the predicate reads a reason in extraReasons as retryable beside (failed) and cannot lock ref, PushWithRetry passes none, and only PushWithRetryReasons passes them. In card 6, add `plan:internal/gitrepo#PushWithRetryReasons` to **Uses:** and have tagAndPushArchiveTip call it with `missing necessary objects`. Card 1's tests pass the reason explicitly; a row asserts PushWithRetry alone does not retry that line. Checked with validate-plan --overlay; re-signing PushWithRetry itself is refused by caller-uncovered."
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/13-remove-follow-up.md (Edit internal/fabriccli#removeFields), internal/fabriccli/fabric.go:1151,1173
    summary: removeFields takes a RemoveResult and a SessionResult and pairteardown.Result.Warning sits on neither, so the card's warning never reaches lym fabric remove's envelope as written
    fix: In card 13, replace `internal/fabriccli#removeFields` with `internal/fabriccli#removeFields` -> `func removeFields(r fabricengine.RemoveResult, session pairteardown.SessionResult, warning string) map[string]any` and add `internal/fabriccli#runRemoveWithFlag` under **Edit:**; its Intent says runRemoveWithFlag passes res.Warning and removeFields adds `warning` when non-empty. Checked with validate-plan --overlay.
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/10-batten-resume-readopt.md (Edit list), internal/battencli/wire.go:200-230
    summary: The refusal text and doc of taskWorktreeLocation and presentTaskWorktreePath say batten never re-creates a pair and send the operator to restore it by hand or delete the run directory, which card 10 makes false
    fix: In card 10, move `internal/battencli#taskWorktreeLocation` from **Uses:** to **Edit:** and add `internal/battencli#presentTaskWorktreePath` under **Edit:**. Its Intent rewrites both doc comments and the refusal to name `lym batten run <slug>`, whose resume re-adopts a live pair, in place of the hand restore and the run-directory deletion, and updates the assertions of TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal that pin "Resolve it by hand". Checked with validate-plan --overlay.
  - id: F10
    severity: LOW
    class: design
    origin: lens:affected-set
    location: _lym/plan/05-teardown-lock-add-discard-push.md (Intent, PushPairAnchored bullet), internal/loomcli/landingdeps.go:329
    summary: The new no-op push under an exclusive hold also reaches the landing's PushTask, which reads a nil return as the task pair pushed, and the card names no bound for that consumer
    fix: "In card 5's PushPairAnchored bullet, name loomcli's landing PushTask (landingdeps.go:329) among the callers. State its bound: only a Remove, discard or sweep of the same slug can hold the lock exclusive while that slug lands. Either show that none of them can run during the landing, or have PushPairAnchored return an error wrapping ErrPairPushRunning instead of nil when it skipped, and keep commit-records' and lym fabric push's warn-and-continue by having those two callers treat that sentinel as a skip."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/05-teardown-lock-add-discard-push.md (Edit internal/fabricengine#PushAnchored)
    summary: Card 5 says commit-records and lym fabric push run PushAnchored, but PushAnchored has no production caller, so its lock and test rows guard a path no entry point reaches
    fix: In card 5, drop `internal/fabricengine#PushAnchored` from **Edit:**, drop the PushAnchored assertions from the test bullets (the "PushAnchored likewise" clause and the prime records-side row), and reword the bullet to name PushPairAnchored as the synchronous push commit-records, lym fabric push and the landing run. Checked with validate-plan --overlay.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/03-add-drops-stale-tracking.md (Test, config.lock row)
    summary: The config.lock row reaches no failure, because DropRemoteTracking writes config only when the branch has upstream config and the row sets only a stale tracking ref
    fix: In card 3's third test row, also set `branch.<weft>.remote` and `branch.<weft>.merge` on the records repository's weft branch before planting config.lock, so the drop's config write fails.
  - id: F13
    severity: LOW
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/01-gitrepo-lease-and-retry.md, _lym/plan/02-gitrepo-branch-push-and-tracking-drop.md, _lym/plan/06-remove-archive-order.md, _lym/plan/08-cleanup-teardown-lock.md
    summary: The cards update the doc comments of the symbols they change but miss other doc text their changes falsify, a method gap rather than one slip
    fix: "Card 1: add `internal/gitrepo#ErrLeaseRejected` under **Edit:**; its Intent also updates the pushretry.go file header and the lease bullet at internal/gitrepo/doc.go:69 to name DeleteRemoteBranchLeased. Card 2: its Intent also adds PushBranch and DropRemoteTracking to the push.go file header's list. Card 6: add `internal/fabricengine#Topology.Remove`, `internal/fabricengine#archiveTagPushDelays` and `pattern/PATTERN-fabric-git.md` under **Edit:**; its Intent rewrites archiveWeftTip's whole doc comment (branch push, up to two tags, weftArchive), Topology.Remove's archive and teardown-lock sentences, Topology's teardownWait field doc (Add and the discard wait too), archiveTagPushDelays' sentence that only the pair's detached pushes wait on the lock, and the PATTERN-fabric-git background's teardown bullet (the weft branch push before the tag). Card 8: move `internal/fabricengine#RemoteWarpBranchEntry` from **Uses:** to **Edit:** and update its Reason and Error field docs for a kept candidate. Checked with validate-plan --overlay together with F7, F8, F9 and F11."
---

The plan carries D1–D8 and every Constraint, its card order is sound, and no card edits this task's board entry or files a task.
One PATTERN-test-speed break makes it BLOCKING.
The other findings are guards that recur unguarded, D2's closed set leaking transient answers that would bring back the endless failing pass, and wiring and doc gaps.

Constraint conflicts, settled for the constraint:
- `d5-no-prose-test` departs from D5's acceptance criterion because `PATTERN-test-speed` says no test pins a stencil's static prose. The plan is right and nothing is flagged.
- `d3-sites-outside-fabric` leaves fabric-worded ways forward (`FailedPushWayForward`'s gone-branch text, `lockPairTeardown`'s re-run, the gate's local-only refusal) unchanged, departing from D3's "every stop or refusal". `PATTERN-fabric-isolation` forbids fabric reading a batten record, so this departure is right for the texts fabric words.
  F6 is the one place outside fabric where batten surfaces such a text and can append the follow-up itself.

### [BLOCKING] Card 10 plans tier0 rows that spawn git
**Location:** `_lym/plan/10-batten-resume-readopt.md`, test bullet for `TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal`; `internal/battencli/wire_test.go:685-690`.
**Issue:** The test is `tiergate.Require(t, 0)`, and its own comment says it "never spawns git".
`taskWorktreeGone` stats the path and then resolves it through `lymcwd.ResolveWorktree`, which runs git.
So the plain-file, empty-directory and resolving-worktree rows all spawn, and the last needs a real worktree from a barred fixture.
`PATTERN-test-speed` ("`tier0` tests spawn nothing") is a listed constraint.

### [MEDIUM] The records-branch refusal of D2's closed set is not reached when the task branch comes from origin
**Location:** `_lym/plan/04-add-adopt-only.md`; `internal/fabricengine/add.go:341-345`, `add.go:493-498`.
**Issue:** `refuseAdoptLocalCheckedOut` runs only under `opts.AdoptLocal && warpAdopted`.
A D1 re-adopt whose task branch exists only on origin, while the local weft branch is checked out in another records worktree, skips the refusal and fails at `containedWorktreeAdd` with a plain error.
Card 12 reads that error as transient, so dobby retries the pass for ever: the failure the task exists to end.

### [MEDIUM] A reused records worktree blocks Add's fast-forward of a behind weft branch
**Location:** `_lym/plan/04-add-adopt-only.md`; `internal/fabricengine/add.go:468-473`, `remoteleftover.go:131-154`.
**Issue:** `probeLiveLocalWeft` sets `fastForwardTo` for a local weft branch behind origin, and Add then fetches into `refs/heads/<weft>`.
git refuses to fetch into a branch any worktree has checked out, and the reused records worktree has it checked out.
The error is outside D2's closed set, so the pass fails on every retry.

### [MEDIUM] Card 6 compares tips before the origin tip is local
**Location:** `_lym/plan/06-remove-archive-order.md`, steps 1–3; `internal/fabricengine/remoteleftover.go:221-230`.
**Issue:** Step 2 decides whether origin's tip descends from the local tip before step 3 fetches that tip.
When another clone moved origin, its tip is not in the records repository, so `merge-base --is-ancestor` fails.
The archive fails, and so does the removal: the diverged and ahead rows the card plans cannot pass as ordered.

### [MEDIUM] The sweeps' lock comes after the state it guards was read
**Location:** `_lym/plan/08-cleanup-teardown-lock.md`; `internal/fabricengine/cleanup.go:131-183`, `cleanupremotewarp.go:89-112`.
**Issue:** `Cleanup` lists the live warp branches, and `cleanupRemoteWarp` lists the candidates, the checked-out map and the tips, all before the loop that takes the per-slug try lock.
An `Add` that finishes between the two goes unseen.
In the remote warp sweep, a landed task branch that a D1 Land re-adopt just tracked keeps its leased tip, so it is deleted from origin.
The card's claim that no new pair's `Add` interleaves with the deletion does not hold.

### [MEDIUM] Batten's resume refusal names `lym fabric remove` without D3's follow-up
**Location:** `_lym/plan/13-remove-follow-up.md`; `internal/battencli/resumereconcile.go:126-127`, `internal/fabricengine/originreconcile.go:333-336`, `internal/fabricengine/failedpush.go:228-236`.
**Issue:** A gone branch on a run's pair gets `FailedPushWayForward`'s `PushFailureGone` text, which names `lym fabric remove`.
Batten's `stoppedRefusal` quotes it in the resume refusal of a batten run, the stop the board entry's 2026-11-03 case followed.
D3 requires that stop to name the cancel or the Land way forward after the removal.
The shared decision exempts only texts fabric words, yet this refusal is battencli's own and can append `BattenFollowUp`.
No `Added after Discussion` entry records the drop.

### [MEDIUM] Card 1 widens the retry beyond the archive-tag push
**Location:** `_lym/plan/01-gitrepo-lease-and-retry.md`, Intent item 1; `internal/gitrepo/push.go:81-128,154-173`.
**Issue:** D4 step 4 licenses a retry on "missing necessary objects" for the archive-tag push only.
Card 1 changes the shared predicate, so every plain push retries that line too.
In `PushRebaseFree`, an exhausted retry wraps `ErrPushRetriesExhausted` where today the line maps to `ErrPushRejected`.
In `Push`, the line no longer reaches the `pull --rebase` recovery.
Both are behaviour changes the answer key does not license.

### [MEDIUM] The removal warning cannot reach the envelope through `removeFields`
**Location:** `_lym/plan/13-remove-follow-up.md`; `internal/fabriccli/fabric.go:1151,1173`.
**Issue:** `runRemoveWithFlag` calls `removeFields(r, res.Session)`, and neither argument carries `pairteardown.Result.Warning`.
The card lists `removeFields` with no re-sign arrow and does not list its caller.

### [MEDIUM] The gone-pair refusal text says batten never re-creates a pair
**Location:** `_lym/plan/10-batten-resume-readopt.md`; `internal/battencli/wire.go:200-230`, `wire_test.go:707`.
**Issue:** `presentTaskWorktreePath`'s refusal is reached when the task worktree disappears mid-run.
It tells the operator that batten does not re-create a pair, and to restore it by hand or delete the run directory, which discards work.
After card 10, `lym batten run <slug>` re-adopts the pair, so the text sends the operator to a destructive step that is no longer needed.

### [LOW] The landing's push meets the new no-op push with no stated bound
**Location:** `_lym/plan/05-teardown-lock-add-discard-push.md`; `internal/loomcli/landingdeps.go:328-331`.
**Issue:** The card names commit-records and `lym fabric push` as the callers that now push nothing under an exclusive hold.
It misses the landing's `PushTask`, which reads a nil return as the task pair pushed.
The attack-surface rule wants the card to state what this lets through and what bounds it.

### [LOW] `PushAnchored` has no production caller
**Location:** `_lym/plan/05-teardown-lock-add-discard-push.md`; `internal/fabricengine/pushanchored.go:82`.
**Issue:** The only production callers are of `PushPairAnchored`: `loomcli/wiring.go:214`, `loomcli/landingdeps.go:329` and `fabriccli/weft_verbs.go:295`.
The card's premise is wrong, and its `PushAnchored` rows test a path no entry point reaches.

### [LOW] Card 3's config.lock row cannot fail
**Location:** `_lym/plan/03-add-drops-stale-tracking.md`, third test row.
**Issue:** Card 2's `DropRemoteTracking` touches config only when upstream config is set.
The row sets only a stale tracking ref, so the drop succeeds and the asserted error never comes.

### [LOW] Doc text falsified outside the cards' own symbols
**Location:** cards 1, 2, 6 and 8; `internal/gitrepo/pushretry.go:2`, `internal/gitrepo/doc.go:69`, `internal/gitrepo/push.go:1,20`, `internal/fabricengine/remove.go:134-137,156`, `internal/fabricengine/archive.go:21,47-57`, `internal/fabricengine/cleanupremotewarp.go:25-28`, `pattern/PATTERN-fabric-git.md:30`.
**Issue:** Each card rewrites the doc of the symbol it changes, but not the file headers, sibling docs or background files that state the old behaviour.
These are the lease sentinel's callers, the retry reasons, the push-surface list, Remove's archive and lock wait, the teardown wait's waiters, the archive step's single tag, a kept candidate's `Reason`, and the background's order of the teardown.

## Rejected

- affected-set 5, the `Skipped` entry missing from cleanup's terminal summary: `fabric.go:1032-1055` computes the exit-code tallies of failures and attempts, where a skipped slug is correctly neither, and the JSON entry carries `skipped` as D7's report.
- repo-invariants 5, the `teardownlock.go` header and `TeardownLockPath` doc: card 5's Intent already rewrites both and names the cleanup sweep among the exclusive holders. The `redundant-file-target` check bars a file target beside its members.
- Handler check: the declared `PATTERN-slug-remote-order` text says "each pair push", but the one synchronous code-side push outside the lock, `Fabric.PushBranch` (`PushWarpRebaseFreeAt`), has one production caller, `landingshed/finalize.go:619`, which pushes the parent branch, not the slug's pair.
- Handler check: `TestPlanPass`'s new `failed` row shares the `run.State != runStateRunning` branch with `HaltedGivesNone`. It is a row, not a new test function, and it pins D2's acceptance criterion, so it is no test-economy finding.
