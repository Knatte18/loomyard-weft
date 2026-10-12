---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/21-status-lock-wait.md
    summary: No card makes a parent lock wait count as activity for the loop's step watchdog (D5), so a Merge-In waiting longer than step_idle_timeout_min is killed as idle.
    fix: In card 21, add `internal/loomcli#agentActivityFor` to the `**Edit:**` list and create `plan:internal/loomcli#TestAgentActivity_LiveLockWaitReadsAsActivity` -> `func TestAgentActivity_LiveLockWaitReadsAsActivity(t *testing.T)`. Extend the Intent so that `agentActivityFor` reads `parentlock.ReadWait(parentlock.WaitRecordPath(loomengine.LoomScratchDir(loc)), parentlock.ProcessAlive)` right after the verify marker, and a live record returns `time.Now(), true, nil` as a running verify does, while an absent record or one whose writer is dead falls through to the shuttle reads. The new test asserts that a live wait record reads as activity now and a dead writer's record does not. Card 21's `Uses:` already names `ReadWait` and `WaitRecordPath`, and its Edit list already holds the test budget.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-fabric-parent-reads.md
    summary: FastForwardPair calls fabricengine Pull, which fetches again and runs its rewrite-reconcile arm (hard reset or re-anchor) on a non-fast-forward upstream, so D1's bound (move the parent pair only when the move is a fast-forward) depends on a read made before Pull's own fetch.
    fix: In card 7, specify that `FastForwardPair` never enters `Pull`'s rewrite path. It fetches the parent pair's code side once, refuses a dirty worktree (`FastForwardDirty`) and a fabric merge record (`FastForwardMergeInProgress`), and refuses with a new `plan:internal/fabricengine#FastForwardNotFastForward` -> `const FastForwardNotFastForward FastForwardRefusal = "not-fast-forward"` when the local head is not an ancestor of the freshly fetched upstream. Only then does it reset to that upstream through the same `ResetHard` arm that `Pull`'s fast-forward branch uses, and it pulls the weft with `PullWeft` as `Pull` does. Drop `internal/fabricengine#Fabric.Pull` from card 7's Uses if it is no longer called, and add a `parenttips_integration_test.go` row in which the upstream is force-pushed after `ReadParentTips` and the pair is left unmoved with the new refusal. In card 12, map `FastForwardNotFastForward` to a Stuck that names `lyx loom resume`, with its own `refusal-spec.md` row and a `mergein_test.go` row.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-parentlock-wait-idle.md
    summary: The parent lock's unbounded queue wait is a new wait of the kind `PATTERN-daemon-wakeups` guards (the land-lock and prime-lock waits are listed), yet no card adds its file to the guard's declared list in cmd/lyx/daemonwakeups_test.go or to the background's What counts list.
    fix: In card 2, state that `Store.Acquire` lives in `internal/parentlock/acquire.go`, add `cmd/lyx/daemonwakeups_test.go` and `pattern/PATTERN-daemon-wakeups.md` to the `**Edit:**` list, and extend the Intent so that the declared unbounded-wait file list gains `internal/parentlock/acquire.go`. The background's What counts list gains the parent lock wait in `internal/parentlock/acquire.go`, which polls from one second doubling to thirty.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/17-merge-in-row.md
    summary: Card 17 bundles the new rubric stencil loom-rubric-merge-review, which is an independently reviewable unit with its own registry, marker allowlist and TestLoomRubrics row, into the recipe row swap.
    fix: With `lyx loom insert-card`, insert a card after card 16 that creates `contracts/stencils/loom/loom-rubric-merge-review.md` and `plan:contracts/stencils#LoomRubricMergeReview`, and edits `contracts/stencils#entries`, `contracts/stencils#rubricMarkerAllowlist` and `contracts/stencils#TestLoomRubrics`, carrying card 17's rubric paragraph as its Intent. Remove those five targets and that paragraph from card 17, and add `plan:contracts/stencils#LoomRubricMergeReview` to card 17's `**Uses:**`. The rest of card 17 stays one card, since Publish losing its merge-in, the recipe rows, the loomcli wiring and their test glyphs must land in one commit.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/19-guarded-landing-rows.md
    summary: Producer tests in cards 12 and 19 re-pin behaviour that lower-layer tests on earlier cards already cover; the lock wait, the per-case moved-parent table and the engine's reroute cap are each tested again at the producer.
    fix: "In card 19's `guardedrows_test.go` list, drop the item in which a fourth reroute is the engine's cap halt, which card 3's `reroute_test.go` covers. The producer test never runs the engine, so it cannot reach the cap. Reduce Publish's reroute to one moved case, because card 11's `parentguard_test.go` `moved` table already covers the upstream push and the local commit. In card 12's `mergein_test.go` list, drop the case in which a lock held by another run makes the call wait, then proceed, which card 11's `parentguard_test.go` `ensure` wait case and card 2's `acquire_test.go` cover; keep only an assertion that the lock is held before `syncParent` runs."
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/07-fabric-parent-reads.md
    summary: Card 7 prescribes the FastForwardMergeInProgress refusal mapping but names no test that reaches it; card 12 only feeds that value through a fake parent puller.
    fix: In card 7's `parenttips_integration_test.go` description, add a row that leaves a fabric merge record on the parent pair (a conflicted fabric merge left unresolved) and asserts that `FastForwardPair` returns `FastForwardMergeInProgress` with a nil error and the pair unmoved.
  - id: F7
    severity: LOW
    class: design
    location: _lyx/plan/01-parentlock-store.md
    summary: Card 1 introduces the ForceRelease and ReleaseAll escape hatches, which free a record whatever its owner, but its Intent states no bound for them; only card 24 bounds the verbs that call them.
    fix: "In card 1's Intent, after the `ForceRelease`/`ReleaseAll` bullet, add: Bound: a force release removes the record only and never stops the owning run or touches its tickets, and the released run reacquires at its next lock-taking row, where the moved-parent check runs."
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/plan/16-recipe-merge-seams.md
    summary: Card 16 says card 17's builder reads Env.MergeReviewOff, but card 18's applyMergeReviewOff in New is the reader.
    fix: In card 16's `Env` bullet for `MergeReviewOff`, replace "which card 17's builder reads" with "which card 18's `New` reads".
---

### [BLOCKING] The lock wait is not watchdog activity

**Location:** `_lyx/plan/21-status-lock-wait.md`

**Issue:**
D5 says Merge-In's lock wait, and each D9 row's wait, "counts as activity for the loop's idle watchdog".
The watchdog in `internal/shedverbs/loopwatchdog.go` kills a step that shows no trace write and no agent activity for `step_idle_timeout_min`, which defaults to 60 in `internal/loomengine/template.yaml`.
`internal/loomcli/looparm.go`'s `agentActivityFor` treats a live verify marker and an unheld shuttle wait as activity now, and nothing else.
No card edits it: card 2's `Acquire` sleeps without logging, card 11's `OnWait` writes only the wait record and the pane mark, and card 21 feeds the record to `lyx loom status` alone.
A second run therefore sits at Merge-In for an hour behind a slow first run and is killed as idle.
That breaks the acceptance criterion "the second waits at Merge-In in queue order … and proceeds once the first's lock is released".

### [MEDIUM] FastForwardPair's bound rests on a stale read

**Location:** `_lyx/plan/07-fabric-parent-reads.md`

**Issue:**
Card 7 wraps `Fabric.Pull`, and card 12 calls it only after `ReadParentTips(true)` has seen the parent behind its upstream.
`Pull` (`internal/fabricengine/pull.go`) fetches again.
On a non-fast-forward upstream with no unpushed local commits, it sets `RewriteDetected` and hard-resets or re-anchors the warp.
`TestPull_ReanchorsSingleCommitBack` pins that behaviour.
An upstream force-pushed between card 12's read and the pull therefore rewrites the parent pair under the lock.
D1's bound says the fast-forward runs "only when that move is a fast-forward", and card 7's Intent states that guarantee without enforcing it inside the function that moves the pair.

### [MEDIUM] The new unbounded wait escapes the daemon-wakeups guard

**Location:** `_lyx/plan/02-parentlock-wait-idle.md`

**Issue:**
`pattern/PATTERN-daemon-wakeups.md` lists the unbounded waits it binds, among them batten's land-lock wait.
`cmd/lyx/daemonwakeups_test.go` scans a declared file list that includes `internal/battenshed/land.go` and `internal/battenshed/primelockwait.go`.
Card 2's queue wait has no time limit and is the same kind of loop, and the decision record's constraint list names `PATTERN-daemon-wakeups` for it.
No card adds the parentlock file to the guard or to the background, so the guard's sub-second scan never covers this wait.

### [LOW] Card 17 bundles a separable rubric stencil

**Location:** `_lyx/plan/17-merge-in-row.md`

**Issue:**
Card 17's Edit list spans `loomshed`, `shedrecipe`, `loomrecipe`, `landingshed`, `loomcli`, `contracts/stencils` and `refusal-spec.md`.
Most of that must be one commit: Publish loses its merge-in in the same commit that Merge-In enters both recipes, or a run lands with no merge-in.
The loomcli guards (`isLandingProducer`, `ownMergeInLeftover`) and the recipe and Publish test glyphs follow the code they change.
The rubric stencil is different: it is a prose artifact with its own registry entry, marker allowlist row and `TestLoomRubrics` row, and it can land before the rows reference it.
Card 14 already gives the rubric fill its `merge_review` marker.

### [LOW] Producer tests duplicate lower-layer coverage

**Location:** `_lyx/plan/19-guarded-landing-rows.md`

**Issue:**
Card 19's `guardedrows_test.go` plans a case in which "a fourth reroute is the engine's cap halt".
The cap is `shedengine`'s, and card 3's `reroute_test.go` pins it through `Step`.
A landingshed producer test never runs the engine, so the case cannot reach the cap, and it repeats card 3's case.
Publish's two moved cases, an upstream push and a local commit, repeat rows of card 11's `moved` table, when one case is enough to pin the row's `Reroute` wiring.
Card 12's "a lock held by another run makes the call wait, then proceed" repeats card 11's `ensure` wait case, which itself sits on card 2's `acquire_test.go`.

### [LOW] No test reaches the merge-in-progress mapping

**Location:** `_lyx/plan/07-fabric-parent-reads.md`

**Issue:**
Card 7 maps `Pull`'s `*ErrMergeInProgress` to `FastForwardMergeInProgress`.
Its integration test covers only the moved pair and `FastForwardDirty`.
Card 12's `mergein_test.go` gets the value from a fake `parentPuller`, so the mapping branch in `fabricengine` has no named test.

### [LOW] Force release has no stated bound where it is introduced

**Location:** `_lyx/plan/01-parentlock-store.md`

**Issue:**
`ForceRelease` and `ReleaseAll` let a caller free a live owner's record.
`ReleaseAll` is exported for a future `lyx pause` caller that this plan does not contain.
Card 1's Intent says what they let through ("whatever its owner") but not what bounds them.
Only card 24's verb paragraph states the bound, and a later caller of `ReleaseAll` will not read card 24.

### [NIT] Wrong card named as the MergeReviewOff reader

**Location:** `_lyx/plan/16-recipe-merge-seams.md`

**Issue:**
Card 16 says `Env.MergeReviewOff` is read by "card 17's builder", but card 18 adds `applyMergeReviewOff` and applies it in `New`.
This is not an order defect, since card 16 creates the field before card 18 reads it, but the cross-reference is wrong.

## Rejected

- Card order: cards 12, 14 and 16 use only targets that earlier cards create. Card 12 needs cards 7, 8, 10 and 11, card 14 needs cards 10 and 13, and card 16 needs cards 10, 13, 14 and 15. Card 16 creates `Env.MergeReviewOff` and card 18 reads it. `NameMergeIn` comes from card 17, and only card 22 uses it. No card depends on a later card; the one wrong cross-reference is F8.
- D14 fidelity: card 1 edits `pattern/PATTERN-told-geometry.md` to list `parentlock`, and card 11 edits `pattern/PATTERN-wait-mark-display-only.md` to add the lock-wait label. Card 19 is the right home for the `PATTERN-parent-lock` entry, because the invariant only becomes true once Publish, PR-Gate and Finalize take the lock. Before card 19, Finalize still lands without it. Not a defect.
- Decisions D1-D13 and the Constraints map to cards:
  - D1: cards 8, 12 and 17.
  - D2: cards 13 and 15-18.
  - D3: cards 14, 16 and 17.
  - D4: card 1.
  - D5: cards 5, 11, 19 and 20. The gap is F1.
  - D6: cards 10, 12, 16 and 17.
  - D7: cards 2, 6, 9 and 20. Card 9's template change goes through `hubreconcile.Ensure`; `landing` is `HubWide` in `configreg` and is loaded from `fabricengine.BoardDir`.
  - D8 and D10: card 23.
  - D9: cards 3, 4, 11, 17 and 19.
  - D11: cards 1 and 24.
  - D12: cards 2, 11, 21 and 22.
  - D13: card 25.
  - D5's cancellation path: card 11 returns a cancellation during the wait as an error, which matches the land-lock wait in `internal/battenshed/land.go`.
  - Scope: card 3's `Halt` flag is licensed, because it is how D4's and D5's halts stay in place at PR-Gate, a row with an `on_stuck` route. No card draws scope from the carry-over block.
- Removing Publish's `CommitStatus` (card 17) does not leave `publish_verify` on a dirty tree. The status file is committed on the records side through `CommitAnchoredPaths`, and `verifytree.DirtyPaths` reads only the code worktree, where `_lyx` is not tracked.
- Reachability: each refusal or Stuck exit has a named input and test that no upstream check rejects first:
  - Card 12's five parent-sync Stucks.
  - The paused and unreadable lock halts in cards 2, 11, 12 and 19.
  - The reroute cap and the missing `on_reroute` wiring fault in cards 3 and 4.
  - Card 24's verb refusals.
  - The constructor nil guards in cards 12 and 19 are fail-loud wiring guards.
  
  The one gap is F6.
- Attack surface:
  - Card 18 bounds `skip_merge_review`.
  - Card 24 bounds the release verbs.
  - Card 20 bounds the idle release and card 23 bounds the backstop.
  - Card 3 bounds the reroute and `Halt`.
  - Card 14 bounds `BlockingEscalates`.
  - Card 22's retry still matches the halting row.
  - Card 12 states the fast-forward's bound.
  
  The gaps are F2 (the bound is not enforced) and F7.
- Member glyphs and re-sign arrows: cards 8, 11, 14, 18 and 22 carry arrows where the Intent changes a signature. `NewPublish`, `Publish.Call`, `NewPRGate`, `NewFinalize`, `Finalize.pushParent`, `waitForChild`, `specFor` and `loopLockOwner` keep their signatures. Every Edit card lists member glyphs rather than bare Go file paths.
- Test economy:
  - Card 5's two new loop tests pin hook behaviour that `TestLoop_WatchdogKillsAnIdleChild` does not cover.
  - Card 21's `TestParentLockWaiting` follows `reviewstatus_test.go`'s one-test-per-waiting-kind shape (`TestVerifyWaiting`, `TestShuttleWaiting`).
  - Card 18's test covers a rewrite that `TestReviewBudget_AppliedByNewAndRouting` does not.
  - Card 22 extends the existing tests.
  
  The duplicates are F5.
- Verify coverage: the repo has one module besides a testdata fixture module. `go test ./...` and `go test -tags integration ./...` run every targeted package, including `contracts/stencils`, `internal/loomcli`, `internal/battenshed` and `internal/battencli`. The section's order matches the plan spec's verify model.
