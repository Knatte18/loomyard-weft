---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:127
    summary: "D5 re-applies Bolt entry changes through the board's normal write validation while D4 has the reconcile hold board.lock, and that write path takes board.lock itself, so the premise cannot hold as written; the unit of a refused change is also undecided"
    fix: "In D5's Bolt bullets (the by-entry replay and the conflict paragraph), state that the Bolt entry merge runs inside the board.lock and .gitrepo-push.lock its caller already holds and takes neither itself, spawns no detached sync, and applies the board's validateWrite and checkRunLocks checks over a Store loaded from origin's board.json, which means a boardengine entry point beside boardCriticalSection and never a call to it. Name the unit of refusal: each entry change or design-file change is validated and applied on its own, a refused change is set aside with the refusal reported, and a later kept change that depends on a refused one (a depends_on naming an entry the refused change added) is refused too and reported, so the set-aside tag carries exactly the refused changes and origin's side is kept for them alone. Replace 'such as the run lock ... or a depends_on' with the board's refusal classes (run lock, dangling or cyclic depends_on, isolated and kind rules, unknown label) so the plan writer need not infer the list, and add an Acceptance criterion where one commit changes a run-held entry and an unrelated entry, and only the run-held change is set aside."
  - id: F2
    severity: BLOCKING
    class: decision
    location: _lyx/discussion/decision-record.md:162
    summary: "D6's one case that ends a busy driver cannot end a driver whose pair worktree is dirty, because the removal refusal probe runs before the driver is ended and cleanup has no force, so the way forward D8 names fails for the usual paused driver and the bound on a forced teardown is undecided"
    fix: "In D6's paragraph on the case that ends a busy driver (and the Bound sentence after it), decide what happens to a dirty task worktree or dirty records outside the pathspec of an ended pair held by a wake-written pair pause entry. pairteardown's EndSession runs the quiet wait, then t.refusal(req.Slug, req.Force), then the loop kill and session end, and RemoveRefusal refuses a dirty task worktree without force, so for a driver paused mid-step the probe refuses first, the driver is never ended and the pair pause entry is never lifted. Either state that this case passes force for dirtiness alone (never for a merge in progress) and what that discards, with the discarded uncommitted paths reported and kept under the local set-aside tag, or state that the case ends the driver and lifts the entry first and then reports the dirtiness refusal with its own way forward; whichever is chosen, bound it to an ended entry holding a wake-written pair pause entry and update D8's busy-flag notice and the Acceptance criterion 'After wake, lyx cleanup --apply removes the ended pair' to cover a dirty worktree."
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:180
    summary: "D8's gate exemption covers only the hub pause entry the daemon's request owns, but a request that found a live hub entry owns none, so the wake reconcile then waits at the gate with no deadline and nothing says so"
    fix: "In D8's wake bullet, add the case where the hibernate request owns no hub pause entry. hibernateengine's pause sets Added only when the entry is new (AlreadyPaused skips it), and confirmation passes on any live hub entry, so a hub pause that stood before the request is the entry the machine woke under, and no owned entry exists to pass. State either that the gate exemption names the live hub entry found at the reconcile's start by its reference whether or not the request owns it, with the same bound (the daemon's process, the reconcile's duration, that entry alone) and with the lift still limited to an owned entry, or that the wake reconcile is skipped and the notice says why. Add an Acceptance criterion for wake under an operator's hub pause. The same gap covers an owned entry lifted meanwhile with another hub entry added."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:64
    summary: "D3 step 2 reads 'removed from origin' from two anchors that both collapse to origin's tip once local Bolt was fast-forwarded by any earlier pull-first, so a pruned entry then reads as never ended"
    fix: "Replace the anchor sentence in D3 step 2 with evidence that does not depend on local Bolt's state: an entry is removed from origin when it is absent from the fetched origin Bolt tip and present in a commit of that tip's own history (its commits are all origin's, so an entry only local Bolt holds never matches), read for the one slug at issue. Say the same wherever D5 (line 113) and D10 (line 225) reuse the rule. As written, the tracking ref 'as it stood before the fetch' is gone after any earlier fetch and the merge base of local Bolt and origin's tip is the tip itself once Bolt fast-forwarded, which Bolt.PullThenCommitWritten does on every pull-first write, so an entry pruned on origin before that fast-forward is absent from both anchors and its pair is not ended. Add an Acceptance criterion where Bolt is fast-forwarded past the removal before the reconcile runs."
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:86
    summary: "D4 defines rewound and rewritten by conditions that overlap (a branch whose local commits all lie in earlier reflog values satisfies both) and gives the two different ways forward without saying which wins"
    fix: "In D4, make the two outcomes disjoint. State which observable separates them (for example rewound when origin's head is an ancestor of an earlier reflog value, which ResetPairCode's leased update produces, and rewritten when it is not) or state the precedence when a branch satisfies both. Rewound's test is 'an earlier value is not an ancestor of origin's head and the local tip holds commits of that value origin's head lacks', and rewritten's is 'no unpushed commit, every local commit reachable from one of the reflog values rewound reads', so every rewritten branch is also rewound. The ways forward differ (ResetPairCode's own rewind or ours merge against lyx fabric pull's anchor walk), so the Acceptance criteria need one case for each outcome."
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:153
    summary: "D6 and D4 say the reconcile never waits on a records write lock, a push lock or a quiet wait, but the removal D6 delegates to waits on the weft write lock, 30 seconds on the teardown lock a parked push holds, and up to two minutes on the loop and run locks"
    fix: "In D6 step 1 and in D4's Bound sentence, state each wait the teardown still makes and set it. Remove's commitPendingRecords takes the weft write lock with a blocking acquire (CommitRecordsPaths in commitweftpaths.go), Remove's lockPairTeardown waits up to StatusPushLockWait (30 s) for a push of the pair that holds the teardown lock shared for its whole run, and a parked push holds it for the whole pause, so at wake each ended pair with a parked push costs 30 s before it is reported busy. EndSession also waits up to runLockWait (2 min) on the loop lock and the run lock once the driver is judged quiet. The local mode should take the teardown lock and the weft write lock by one try-acquire and report the pair busy when either is held, and D4's Bound sentence should list the teardown lock beside the other locks. If a wait is kept on purpose, name its value and why it is bounded."
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:225
    summary: "D10's gone path mails the orch when the entry is not yet done on origin's Bolt, and the landing machine settles the board claim after the removal that deletes the branch, so a short window of spurious mail exists with no stated bound"
    fix: "In D10's gone paragraph, state what a gone push does when the fetched origin Bolt tip shows the entry neither done nor removed. pairteardown's RemovePair marks a landed run's entry done only after Topology.Remove, and Bolt's push is detached, so origin can lose the branch before it gains the done mark. Either class that case as the push's own transport-like backoff retried once after a delay (with the existing backoff bound) before the failed-push record and mail, or state that the mail is accepted and the record is removed by the next reconcile that finds the entry ended."
---

### [BLOCKING] D5 Bolt merge reuses a write path that takes the lock the reconcile already holds

**Location:** `_lyx/discussion/decision-record.md:127` and `:132`, against `internal/boardengine/board.go:54` and `internal/boardengine/runlock.go:69`.

**Issue:** D4 has the reconcile hold Bolt's `board.lock`, then `.gitrepo-push.lock`, while D5 re-applies each kept Bolt commit through the Bolt entry merge "through the board's normal write validation".
The board's validation lives in `boardCriticalSection`, which opens `writeLockFile` (the same file as `fabricengine.BoardWriteLockFile`, set at `internal/boardengine/sync.go:27`) with a blocking `AcquireWriteLock`, then saves, renders and spawns a detached sync.
Called under the reconcile's hold it blocks on itself, and the record names no other entry point.
The run-lock check there (`checkRunLocks`) judges the net change of one whole write against the loaded entries, so one refused entry refuses everything applied in that write.
The record says "the local commit is also kept under a set-aside tag" but not whether a refusal sets aside a commit or a change, nor what happens to a later change that depends on a refused one.
The record does say the run lock is never bypassed and that origin's side is kept, so the bound on the outcome is stated; what is missing is the mechanism and the unit.

### [BLOCKING] The way forward for an ended pair with a busy driver fails on a dirty worktree

**Location:** `_lyx/discussion/decision-record.md:162` and `:185`, against `internal/pairteardown/teardown.go:150` and `internal/fabricengine/remove.go:679`.

**Issue:** D8 names `lyx cleanup --apply` as the way forward for an ended entry with a busy flag, and D6 makes that teardown skip `RefuseWhenBusy` so the held driver is ended.
`EndSession` runs the quiet wait, then the refusal probe with `req.Force`, and only then the loop kill and the session end.
`RemoveRefusal` refuses a task worktree with uncommitted changes unless force is set, and cleanup has no force.
A driver the pause interrupted mid-step usually leaves uncommitted changes, so the probe refuses before the driver is ended, the wake-written pair pause entry is never lifted and the parked push stays parked.
`lyx fabric remove --force` does not help, since it passes `RefuseWhenBusy`.
The record's Bound says only that the case ends a busy driver; it does not say what a forced or reordered teardown may discard.

### [MEDIUM] The wake exemption has no case for a hub pause the request does not own

**Location:** `_lyx/discussion/decision-record.md:180`, against `internal/tenderengine/pause.go:126` and `internal/hibernateengine/doc.go` (the pause paragraph).

**Issue:** D8 exempts "the hub pause entry the daemon's own request owns" and bounds the exemption to the reconcile's two ls-remote calls and its fetches, which is a stated bound.
A pause that found a live hub entry owns none, and confirmation accepts any live hub entry, so a machine hibernated under an operator's pause wakes with nothing to exempt.
`pauserecord.Network` waits through `WaitUncovered` on a background context with no deadline, so the daemon would sit in the resuming phase until someone runs `lyx resume`, with no notice and the reconcile run against then-live pairs.
The Acceptance criterion "Wake with the hub pause live completes the reconcile and lifts the hub entry" covers only the owned case.
The record settles what `lyx orch start` and `lyx cleanup` do under an operator's hub pause (they wait at the gate and the wait line names the pause).

### [MEDIUM] The removal evidence for the ended set is lost after an earlier Bolt fast-forward

**Location:** `_lyx/discussion/decision-record.md:64`, with the same rule reused at `:113` and `:225`.

**Issue:** D3 step 2 accepts an entry as removed when it was in "the Bolt tracking ref as it stood before the fetch, or the merge base of local Bolt and origin's tip".
Any earlier fetch moves the tracking ref, and a pull-first that fast-forwards local Bolt makes the merge base the tip, so neither anchor holds the entry.
`Bolt.PullThenCommitWritten` pulls first on every write it runs, so a woken machine whose Bolt was brought level before the reconcile loses the evidence that a pruned entry ever existed on origin.
D10's gone path fetches Bolt and then reads "the ended set D3 step 2's rule reads from that tip", so it inherits the same gap; the directive's premise that the gone path fetches nothing no longer holds, and the fetch it now makes is not the weak point.

### [MEDIUM] Rewound and rewritten overlap

**Location:** `_lyx/discussion/decision-record.md:81` and `:86`.

**Issue:** Every branch that meets the rewritten condition (no unpushed commit, all local commits reachable from earlier reflog values) also meets the rewound condition (an earlier value is not an ancestor of origin's head and the local tip holds commits of it origin lacks).
The record says rewound outranks ahead but is silent on rewound against rewritten, and the two name different ways forward, so `PATTERN-refusal-way-forward`'s test cannot tell which one the reconcile must reach.
The evidence-survival question itself is settled (see Rejected).

### [MEDIUM] The teardown the reconcile calls still waits

**Location:** `_lyx/discussion/decision-record.md:153` and `:106`, against `internal/fabricengine/remove.go:213`, `internal/fabricengine/teardownlock.go:93` and `internal/pairteardown/loop.go:19`.

**Issue:** D4's Bound says the reconcile never waits on a records write lock, a push lock or a driver's quiet wait, and D6 passes a quiet wait of zero.
`Remove` still calls `commitPendingRecords`, whose `CommitRecordsPaths` acquires the weft write lock with a blocking `AcquireWriteLock`, and takes the pair's teardown lock waiting up to `StatusPushLockWait` (30 s), which a detached push holds shared for its whole run, so at wake a parked push holds it for the whole pause.
Once the driver is judged quiet, `EndSession` also waits up to two minutes each on the loop lock and the run lock.
D6 says such a pair "is reported busy", but only after those waits, and the Bound does not list the teardown lock.

### [LOW] A gone push can mail before origin's Bolt shows the entry ended

**Location:** `_lyx/discussion/decision-record.md:225`, against `internal/pairteardown/doc.go` (the board claim paragraph).

**Issue:** The landing machine settles the board claim after the removal, and Bolt's push is detached, so origin can lose the branch before it shows the entry done.
A woken machine's parked push then takes the gone path, finds the entry neither done nor removed in the fetched tip, writes a failed-push record and mails the orch once.
The Acceptance criteria cover only the case where the entry is already done on origin.

## Rejected

- The judge-only dry run moving a tracking ref: not a defect. D8 fetches into no ref (`--refmap=` with no destination, `--no-write-fetch-head`) and states "the dry run moves no ref, tracking refs included", and the hub stores are non-bare so their reflogs are on.
- Evidence of a rewind lost to a fetch outside the reconcile (`Fabric.Pull`, the push retry, Bolt's pull-first, the dry run): not a defect. D4 reads the tracking ref's reflog from this machine's last own push onward, and the Constraints name the reflog expiry limit.
- The run lock being bypassed by the replay: not a defect. D5 says the merge never bypasses the run lock and keeps origin's side with the local commit under a tag; F1 is about the mechanism, not this rule.
- `lyx orch start` and `lyx cleanup` under an operator's hub pause: settled. D8 says both wait at the network gate until the lift and the wait line names the pause.
- A parked push of a run landed elsewhere mailing the orch: settled for an entry already done or removed on origin, by D10's Bolt fetch before judging ended membership; the remaining window is F7.
