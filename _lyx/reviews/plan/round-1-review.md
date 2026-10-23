---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/16-batten-pre-drive-check.md:186
    summary: The pre-drive refusals reach callers through shedverbs' step verb without the trailing way-forward clause PATTERN-refusal-way-forward requires; this covers card 16's seed-gone text, card 14's cancelMarkerRefusal and card 6's ErrLockedCheckRefused wrap.
    fix: >-
      In card 16's Intent, end the seed-gone refusal with a trailing clause, worded as `battencli: <slug> has no seed, so its run was cancelled or never started; way forward: run "lyx batten run <slug>" for a fresh start`. In card 14's Intent, end both cancelMarkerRefusal texts with a `way forward:` clause. The live-writer text names how to see the holder, as `way forward: wait for the cancel (pid <pid>) to end; check it with "lyx batten status <slug>", then run the verb again`; the gone-writer text reads `way forward: run "lyx batten cancel <slug>" from the prime to finish it`. In card 6's Intent, state that a LockedCheck error ends in a `way forward:` clause that the ErrLockedCheckRefused wrap keeps trailing. In card 6's new TestStepCmd_BusyTreatment row and card 16's TestPreDriveCheck refusal rows, assert that the reported message ends with the check's `way forward:` clause.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/17-batten-cancel-core.md:231
    summary: TakeBattenLocks is not required to release a lock it took before it reports busy, so card 13's busy-then-retry re-runs the hook while this process still holds the driver lock; the retry then waits out the full bound on its own flock and refuses.
    fix: In card 17's cancelDeps doc for TakeBattenLocks, and in card 18's production wiring of it, require that when the second lock (the run lock) is busy, the seam releases the driver lock it already took before it returns the busy path, so that a busy report holds no lock. In card 13's CancelRequest doc, state that AfterEnd holds nothing when it reports busy. Add a TestCancelRun row in which TakeBattenLocks reports busy once and then succeeds, asserting that the second call is made with no lock held from the first.
    origin: handler
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/17-batten-cancel-core.md:241
    summary: With origin unreachable the cancel reads no owner at all, so it ends this machine's processes of a run that the local Bolt records as another machine's; D2's bound says the offline owner comes from the local Bolt.
    fix: In card 17's step 2, state that on an unreachable origin from either source (the reconcile's Unreachable, or a pull error wrapping boardengine.ErrOwnerOriginUnreachable), the cancel reads Owner from the local Bolt before it goes end-only, and refuses with the cancel's owned-elsewhere wording when another machine owns the slug, with nothing ended. Add TestCancelRun rows for both unreachable sources with a local owner elsewhere, each asserting the refusal and no EndForCancel call.
    origin: handler
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/18-batten-cancel-verb.md:322
    summary: Card 18 introduces the --remove flag but its Intent states only the verb's skip and bound, never what --remove lets through or what bounds it.
    fix: In card 18's Intent, after the verb's skip and bound, add the attack-surface statement for --remove. What it lets through is a forced removal that discards the pair's uncommitted paths and the deletion of an unlanded task branch, locally and on origin. What bounds it is that the local snapshot tag is taken first and a refused snapshot removes nothing; that every deleted tip is held by an archive tag on the warp origin, and a failed tag push deletes no branch; and that D2's reached-parent refusal keeps it from ever removing a pair whose run here has work that reached the parent.
    origin: handler
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/plan/17-batten-cancel-core.md:247
    summary: If an earlier cancel left its marker and the run then reached its parent, a re-run of the cancel refuses at step 3 naming lyx batten run, which refuses on the marker naming the cancel, a loop with no lyx exit.
    fix: Ask the parent to record an Added-after-Discussion decision for the reached-parent refusal when a gone writer's cancel marker is present. The proposed decision is that card 17 takes that marker over and removes it, as step 4 would, before returning the reached-parent refusal, so that lyx batten run can finish the landing. Then carry the decision in card 17's step 3, with a TestCancelRun row for a reached-parent refusal over a gone writer's marker that asserts the marker is removed and nothing else is ended or changed.
    origin: lens:way-forward
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/plan/13-teardown-cancel-phase.md:46
    summary: The spent-bound refusal for a busy records write lock names only the lock path and the cancel again, and gives no way to see or end the holder, which can be the run's own process wedged inside a records commit that the re-run would never end.
    fix: >-
      In card 13's step 1, word the busy-write-lock refusal as driverLockBusyError words its own. It names the lock path and the way forward: find the holder with `lyx tender status` or `ps` (in Task Manager on Windows), end it, then run "lyx batten cancel <slug>" again. TestEndForCancel's busy-write-lock row asserts these names.
    origin: lens:way-forward
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/10-board-runlock-names-cancel.md:5
    summary: Card 10 rewords RunLockedError's way forward, but internal/boardengine/doc.go:148, which describes that message's ways forward as clearing the status, is not on the card and goes stale.
    fix: Add `internal/boardengine/doc.go` to card 10's Edit list. Its Intent rewords the sentence at doc.go:148 to say an abandoned run is ended with `lyx batten cancel <slug>` from the prime, which clears the status. The overlay validation passes this target.
    origin: lens:affected-set
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/06-shed-locked-check-and-precondition.md:207
    summary: stepCmd now maps ErrLockedCheckRefused onto StepBusyKind too, but the Spec field's doc at internal/shedverbs/spec.go:200 still says it is the kind ErrShedBusy maps onto.
    fix: Add `internal/shedverbs#Spec` to card 6's Edit list, and add a line to its Intent rewording the StepBusyKind field doc to name both ErrShedBusy and ErrLockedCheckRefused. The overlay validation passes this target.
    origin: lens:affected-set
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/16-batten-pre-drive-check.md:174
    summary: battenPreStep's doc comment (internal/battencli/arm.go:728-733) states the hook's closed kind mapping, which card 16 changes by mapping a refused seeding to KindBusy, but card 16 does not edit that glyph.
    fix: Add `internal/battencli#battenCLI.battenPreStep` to card 16's Edit list, and add a line to its Intent adding to that doc comment's mapping that a seeding the pre-drive check refuses reports KindBusy. The overlay validation passes this target.
    origin: lens:affected-set
---

### [BLOCKING] Pre-drive refusals carry no trailing way-forward clause

**Location:** `_lyx/plan/16-batten-pre-drive-check.md:186`, `_lyx/plan/14-batten-cancel-marker.md:104`, `_lyx/plan/06-shed-locked-check-and-precondition.md:222`

**Issue:** `PATTERN-refusal-way-forward` binds `internal/shed*`.
Every refusal reachable through a bound module's verbs must name its way forward in a trailing `way forward:` clause, and only refusals that predate the rule keep their own wording (`contracts/specs/refusal-spec.md`).
Card 6 wraps a `LockedCheck` error as `fmt.Errorf("%w: %w", ErrLockedCheckRefused, err)`, and `stepCmd` reports it with the check's own text.
Card 16 maps the seeding-closure refusal through the step verb's `PreStep` hook.
Both are new refusals that leave through `internal/shedverbs`.
The texts they carry are new, and none of them has the clause:

- card 16's seed-gone text, `…; run "lyx batten run <slug>" for a fresh start`;
- card 14's live-writer text, `…; wait for it to end`, which also never says how to see the holder;
- card 14's gone-writer text.

Shed's own busy refusal shows the convention (`busyWayForward`, `internal/shedengine/run.go:132`).
The planned tests assert `errors.Is` and the kind only, never the clause.

### [MEDIUM] A busy batten-lock hook keeps the lock it took across the retry

**Location:** `_lyx/plan/17-batten-cancel-core.md:231`, `_lyx/plan/13-teardown-cancel-phase.md:39-50`

**Issue:** Card 13 retries `AfterEnd` after a busy report and says the caller releases what its hook took "on every return".
Card 17's `TakeBattenLocks` returns "the release or the busy path", with no rule for a driver lock already taken when the run lock reads busy.
Card 18 takes each lock with `lock.AcquireWriteLockWithin` on a fresh handle.
D6 itself notes that fresh `gofrs/flock` handles conflict with the same process's held locks (`internal/battencli/cleanup.go:187-206`, `internal/lock/lock.go`).
So a hook that keeps the driver lock blocks on itself in the retry until the bound is spent, and the phase refuses even though the real holder has gone.

### [MEDIUM] The offline cancel reads no owner

**Location:** `_lyx/plan/17-batten-cancel-core.md:241-243`

**Issue:** D2's bound reads "offline the owner comes from the local Bolt", and `PATTERN-owner-machine` has a resume with origin unreachable read the local Bolt; `reconcileResume` does the same through `refuseResumeUnreachable`.
With the task worktree present, card 17 reads `Owner` only on the absent path.
With the pull unreachable, it does not say `Owner` is still read.
Either way, an unreachable origin goes straight to end-only, which ends this machine's processes of a run the local Bolt records as another machine's.

### [MEDIUM] `--remove` has no stated bound

**Location:** `_lyx/plan/18-batten-cancel-verb.md:320-322`

**Issue:** The rubric's attack-surface rule asks the card that introduces a flag to state, in its `**Intent:**`, what the flag lets through and what bounds it.
Card 18 introduces `--remove`, and its Intent gives a skip and a bound for the verb only.
For `--remove` it just describes what it does.
Nowhere in the card is the forced, discarding removal or the deletion of an unlanded branch tied to its bounds, which are the snapshot, the archive tags and the reached-parent refusal (D5's bound).

### [MEDIUM] The reached-parent refusal and a left marker form a loop

**Location:** `_lyx/plan/17-batten-cancel-core.md:247-253`, `_lyx/plan/15-batten-start-guards.md:144`

**Issue:** Card 17 leaves the marker on every refusal at or after step 5, and on the end-only path.
A live run's Finalize can pass its parent-side merge between an earlier cancel's step 3 and its end.
The re-run of the cancel then refuses at step 3, before its step 4 marker take-over, and names `lyx batten run <slug>`.
That verb refuses on the marker (card 15, step 1) and names `lyx batten cancel <slug>`.
No lyx verb leads out of the loop.
D2 says refusals change nothing, and D3 does not order the two, so this needs a decision through the parent.

### [MEDIUM] The busy write-lock refusal does not say how to reach the holder

**Location:** `_lyx/plan/13-teardown-cancel-phase.md:46`

**Issue:** The records write lock is held across whole fabric operations; `lockWeftForReplay` holds it and then takes the push lock (`internal/fabricengine/pull.go:443-460`).
If the run's own process is wedged while holding it, the cancel times out before `End` and ends nothing, so every re-run refuses the same way.
The refusal-spec counts waiting for a holder as a way forward only when the message says how to see the holder.
This one names only the path.

### [LOW] Board package doc keeps the old run-lock way forward

**Location:** `_lyx/plan/10-board-runlock-names-cancel.md:5`

**Issue:** `internal/boardengine/doc.go:148` describes the `RunLockedError` message's ways forward as clearing the status once `lyx batten status` shows no run holds it.
Card 10 changes that message, but the doc is not on the card.

### [LOW] `Spec.StepBusyKind` doc names one sentinel

**Location:** `_lyx/plan/06-shed-locked-check-and-precondition.md:207`

**Issue:** `internal/shedverbs/spec.go:200-201` says `StepBusyKind` is the kind step maps `ErrShedBusy` onto.
Card 6 maps `ErrLockedCheckRefused` onto it as well.

### [LOW] `battenPreStep` kind-mapping doc goes stale

**Location:** `_lyx/plan/16-batten-pre-drive-check.md:174`

**Issue:** `internal/battencli/arm.go:728-733` lists the hook's closed kind mapping, under which a failed status seed is `KindUnseeded`.
Card 16 makes a seeding the pre-drive check refuses report `KindBusy`, and edits only `battenPreStepUnmarked`.

## Rejected

- lens:guard-reach, BLOCKING: "the prime-side loop's step child keeps batten's run lock because `runproc.End` cannot reach it". This rests on the link the fork left unverified, and that link is false. `planRecords` walks every descendant of a recorded running process, and signals a child that leads its own group through that group (`internal/runproc/end.go:303-331`). The loop registers as `RoleLoop` (`internal/battencli/arm.go:500-504`), so its `lyx batten step` child is reached and ended before card 17 takes the batten locks.
- lens:guard-reach, LOW: "`checkRemoteBranchRequest` refusing `dirtyArchivedTip` is unreachable". Without the explicit case, the new kind would fall through to `checkedOutBranchDirtiness` (`internal/fabricengine/destroy.go:1192`) and pass a remote delete with no archive proof. So the case is a fail-closed wiring guard in the destructive gate, of the kind `refusal-spec.md` exempts, and it mirrors the existing remote-kind refusals in `checkBranchDirtiness`.
- handler, considered: "card 18 calls `DiscardTaskBranch` outside `pairteardown`, against D5's title". D5's body runs the archive-and-delete also when `RemovePair` ran not at all or read `ErrPairNotFound`, and the call removes no worktree after `EndForCancel` has ended everything. So it breaks neither the decision's text nor the declared `PATTERN-pair-teardown`.
- handler, considered: "card 19 adds `goto` to the overview's verb list beyond the license". That corrects the line card 19 is licensed to edit, where `goto` is already a batten verb, and dropping it would leave the doc wrong.
