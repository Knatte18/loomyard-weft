---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/17-shuttle-shell-only-waiting.md:6
    summary: >-
      Card 17 leaves three existing tests that pin the old shell-wait rule out of its targets, so its card gate goes red on tests it never names.
    fix: >-
      In _lyx/plan/17-shuttle-shell-only-waiting.md add three Edit targets after `internal/shuttleengine#TestWait_RecordsEndedShells`:
      `internal/shuttleengine#TestShellWaitMark`,
      `internal/shuttleengine#TestWait_PayloadShellLogsPastWaitMinOnceAndFinishesAtTheNextTurnEnd`
      and `internal/shuttleengine#TestWait_HeldTurnEndNotice`.
      In the card's Tests paragraph state the new expectation of each.
      `TestWait_HeldTurnEndNotice` (hold_test.go), row "a shell outstanding keeps waiting past the bound with no held turn end and no notice":
      an ungated run with an output file missing now sends exactly one held-turn-end notice at the first tick past the bound and still ends OutcomeTimeout at the deadline; rename the row to say so.
      `TestWait_PayloadShellLogsPastWaitMinOnceAndFinishesAtTheNextTurnEnd` (waiting_test.go):
      its `len(notices) != 0` assertion and its message "a shell outstanding is never a held turn end" now expect one notice, the past-wait-min Warn line stays at one, and the run still ends Done at the agent's next turn end; rename it to drop "FinishesAtTheNextTurnEnd" only if the card's wording needs it.
      `TestShellWaitMark` (waiting_test.go):
      rows "a transcript-reported shell shows from the bound", "a payload-reported shell shows from the bound" and "a later event turns it off" cannot hold, because the single shell that crosses the bound now expires the ungated wait in the same tick and clears the waiting list before `syncShellWait` reads it.
      Rewrite the three rows over two shells outstanding at staggered first-seen times, one past the bound and one not, which is the only state in which the `background shells` mark still shows; keep the fork and awaited-shell rows.
      The combined edit was checked with `lyx loom validate-plan --overlay` and reports no new blocking finding.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/17-shuttle-shell-only-waiting.md:50
    summary: >-
      Card 17 rewords some comments and docs of the old shell-wait rule but leaves four that contradict the new rule untouched.
    fix: >-
      In _lyx/plan/17-shuttle-shell-only-waiting.md add Edit targets `internal/shuttleengine#Spec`, `internal/shuttleengine#Run.shellWaitActive` and `internal/websterengine#masterAwaitedShellPrefix`,
      and extend the Docs paragraph with these rewordings.
      `Spec.AwaitedShellPrefixes` (spec.go): it says a prefixed shell only stays out of the wait marker "since every shell holds a gated run's turn end alike"; after D11 a prefixed shell is also what keeps a gated waiting turn end from being an arrival and keeps an ungated wait from expiring.
      `masterAwaitedShellPrefix` (websterengine/runlevel.go): it says the prefix "does not change when a turn end counts"; after D11 it is exactly what keeps the recovery verb's shell holding a gated turn end.
      `Run.shellWaitActive` (waitmark.go): "It is display only: the wait it shows expires nothing" is false for an ungated run, where the wait expires at the same bound; also state that the mark now shows only while shells are outstanding at staggered times, since a single non-awaited shell expires or arrives at the bound it would be shown at.
      `Run.pollEventsTick` doc (wait.go): "A waiting turn end never expires: a shell of either signal and a fork keep the turn waiting however long they run" and the ungated at-once sentence need the same two-rule rewording as the file header and `expiredTurnEnd` doc the card already names.
      The doc.go sentence "stays the display threshold of every other wait" in the card's Intent should say which wait still displays (the staggered-shell wait) instead of implying a general one.
      The extended target list was checked with `lyx loom validate-plan --overlay` and reports no new blocking finding.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-fabric-remove-reconcile.md:36
    summary: >-
      A stopped verdict caused by a branch gone from origin refuses with a way forward that is this same `lyx fabric remove`, which loops.
    fix: >-
      In _lyx/plan/11-fabric-remove-reconcile.md change `removeVerdict` so a stopped entry whose stopping branches of the slug (report.Branches with Slug equal to the slug and a stopping State) all read `fabricengine.BranchGone`, with no Bolt or prime-records branch stopping, returns the empty verdict, today's request, instead of refusing.
      Reason to state in the Intent: the way forward of a gone branch (`FailedPushWayForward` for the gone class, reached through `stopWayForward`) is "`lyx fabric remove` of the pair ... removes the pair and clears this record", so refusing it names the refused verb.
      A gone branch beside any other stopping branch still refuses with that other branch's way forward.
      Add `TestRemoveVerdict` rows: stopped only by a gone code branch of the slug gives the empty verdict; stopped by a gone branch and a diverged one still refuses naming the diverged branch's way forward.
      Add a `TestRemover` row: a gone-branch stop runs today's request with `RemoveQuietWait` and no refusal.
      This edit is card text only and changes no target list.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-fabric-remove-reconcile.md:36
    summary: >-
      Card 11 prescribes a refusal for a stopping branch that reads live, but a run-scope reconcile never produces one, and a test row names the unreachable input.
    fix: >-
      In _lyx/plan/11-fabric-remove-reconcile.md delete from the `removeVerdict` paragraph the clause beginning "when a branch stopping the entry reads live" through "under whose hub entry a live pair reads not live",
      and delete from the `TestRemoveVerdict` rows the row "stopped with a live parent branch names the parent pair and `lyx tender pause` with no slug before the re-run".
      Keep the row "the parent pair's entry of a run-scope report is not read as the slug's".
      Evidence: `originMover.move` returns early for a non-Bolt target whose slug differs from `scope.Run` before it reaches `pairLive`, `pairLive` runs only for a slug other than `scope.Run`, and a Bolt or prime target has the empty slug, so `BranchLive` (set only in `originreconcilemoves.go`) never appears in a run-scope report; `classifyEntries` also attributes a branch only to its own slug's entry.
      This edit is card text only and changes no target list; it touches the same bullet as F3, so apply both to that one paragraph.
  - id: F5
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-runproc-owner-record.md:74
    summary: >-
      TestEnd has no row reaching the group identity check at either phase or the record-level pid-reuse check in the SIGKILL phase.
    fix: >-
      In _lyx/plan/02-runproc-owner-record.md extend the `TestEnd` rows with:
      a recorded leader whose group no live process reports any more gets no group signal;
      a recorded leader whose group id is now reported only by a process with a start time before the recorded one gets no group signal;
      the same two cases between the SIGTERM and the SIGKILL, each getting no SIGKILL group signal;
      a recorded non-leader whose pid reads a start time other than its record's between the SIGTERM and the SIGKILL gets no SIGKILL (the existing reuse row covers a descendant only).
      These are rows in the existing test function, not new tests, and the card text change adds no target.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/10-teardown-skip-readings.md:7
    summary: >-
      runLockHolderPasses takes the slug's registered records but its stated rule never reads them, and the rule says nothing on an unknown holder under the paused pass.
    fix: >-
      In _lyx/plan/10-teardown-skip-readings.md drop the `registered []runproc.Record` parameter from the `runLockHolderPasses` signature,
      drop the Uses lines `plan:internal/runproc#List` and `plan:internal/runproc#Record`,
      drop "the slug's live owner records" from the sentence that lists what `awaitRunLockOrPass` reads,
      and in `TestRunLockHolderPasses` change "a holder with a marker of another registered pid of the slug refuses" to "a holder with a marker of another pid refuses".
      State in the Intent that the paused pass (a live pause entry covering the slug and no parent lock record naming it) applies to a known and an unknown holder alike,
      because `End` and the loop end reach what they can and the driver-lock re-check refuses a holder neither reached;
      add a table row: unknown holder, paused, no parent lock record passes.
      The combined edit was checked with `lyx loom validate-plan --overlay` and reports no new blocking finding.
  - id: F7
    severity: LOW
    class: design
    location: _lyx/plan/10-teardown-skip-readings.md:65
    summary: >-
      For an unpaused run-lock holder the refusal names only retrying once its pid ends, which never succeeds for an ended or foreign run whose driver chain is still live.
    fix: >-
      In _lyx/plan/10-teardown-skip-readings.md change `runLockRefusal` so a refusal with `paused` false names, ahead of the retry, the way forward `lyx tender pause <slug>` and then the removal again,
      and keep "retrying the removal once that pid ends" as the second option.
      Reason to state: the skip also reads true for an `ended` or `owned_elsewhere` verdict with no pause, nothing in the removal pauses that run, and a Go driver holds the run lock for the run's whole life, so the pid does not end;
      a pause passes the holder at the next attempt through the paused pass when no parent lock record names the slug.
      Extend the refusing-row wording assertions in `TestRunLockHolderPasses` so an unpaused known holder and an unpaused unknown holder both assert `lyx tender pause slug`.
---

### [BLOCKING] Card 17 omits three existing tests the new rule breaks

**Location:** `_lyx/plan/17-shuttle-shell-only-waiting.md:6` (Edit list) and `:41-49` (Tests paragraph).
**Issue:** The card changes `expiredTurnEnd` so an ungated run with an output file missing, whose waiting list holds only non-awaited shells all outstanding for `ShellWaitBound`, returns the held turn end and clears `waitingTasks`.
It names two tests to flip and misses three that run exactly that state.
`TestWait_HeldTurnEndNotice` has the row "a shell outstanding keeps waiting past the bound with no held turn end and no notice": an ungated run, output missing, a shell outstanding, 6-minute sleeps against a 10-minute bound; it asserts no notice and `OutcomeTimeout`, and the new rule sends one notice through `notifyHeld`.
`TestWait_PayloadShellLogsPastWaitMinOnceAndFinishesAtTheNextTurnEnd` runs the same shape and asserts `len(notices) == 0` with the message "a shell outstanding is never a held turn end".
`TestShellWaitMark` drives `pollEventsTick` then `syncShellWait` on an ungated fixture with the output file missing; at the step that advances the clock to the bound, `pollEventsTick` now expires the wait and empties `waitingTasks`, so `shellWaitActive` reads false and the rows expecting the mark on fail.
The card's gate would be red on tests outside its target list, and its implementer has no stated intent for what `TestShellWaitMark` should pin once a single shell can no longer show the mark.

### [MEDIUM] Four comments and docs the card does not list still state the old rule

**Location:** `_lyx/plan/17-shuttle-shell-only-waiting.md:50-52`.
**Issue:** The Docs paragraph names `wait.go`'s header, `expiredTurnEnd`'s doc, the `Config` field comment, `ShellWaitBound`, `template.yaml`, the two `doc.go` files and `docs/overview.md`.
Four other texts say the opposite of D11 after the card lands: `Spec.AwaitedShellPrefixes` in `spec.go` ("since every shell holds a gated run's turn end alike"), the doc of `masterAwaitedShellPrefix` in `websterengine/runlevel.go` ("does not change when a turn end counts"), `shellWaitActive` in `waitmark.go` ("the wait it shows expires nothing"), and `pollEventsTick`'s doc in `wait.go` ("A waiting turn end never expires").
The `websterengine/doc.go` rewrite the card orders would then disagree with the const comment two files away.
Also, D11 and the card say `BackgroundShellWaitMin` "stays the display threshold of every other wait", but after the card the `background shells` mark can show only while shells are outstanding at staggered times, so the docs should say that rather than imply a general wait.

### [MEDIUM] A stopped verdict from a gone origin branch refuses with the refused verb

**Location:** `_lyx/plan/11-fabric-remove-reconcile.md:36`.
**Issue:** `removeVerdict` refuses every `stopped` entry with the entry's way forward and then `lyx fabric remove <slug>` again.
For a branch that origin no longer has, `BranchGone` stops its entry (`BranchState.stops`), and its way forward, built by `stopWayForward` through `FailedPushWayForward`, reads "decide whether the run ends: `lyx fabric remove` of the pair ..., run from the prime, removes the pair and clears this record".
After the card, `lyx fabric remove` on that pair refuses naming `lyx fabric remove`, and the retry fails the same way; `ended` does not rescue it, since the entry is neither done nor removed on origin.
Today that removal works, so the card removes the only documented exit for a deleted origin branch short of the board marking the entry done.

### [MEDIUM] The live-parent-branch refusal in card 11 can never be reached

**Location:** `_lyx/plan/11-fabric-remove-reconcile.md:36` and `:47`.
**Issue:** The card says that when a stopping branch reads live it is the parent pair's, and prescribes a refusal naming the parent pair and `lyx tender pause`, with a `TestRemoveVerdict` row.
In `fabricengine`, `BranchLive` is set only inside `originMover.move`, after an early return for any non-Bolt target whose slug is not `scope.Run`, and only for a slug other than `scope.Run` with a non-empty slug.
A run-scope reconcile therefore never marks any of its targets live, and `classifyEntries` gives a branch only to its own slug's entry, so no entry of the slug carries the "no step now" way forward.
The refusal has no input that reaches it; the row can only be written by hand-building a report the reconcile never returns.
The round-1 fixer report (F9) grounded the clause on `originreconcilemoves.go:379`, `target.slug != m.scope.Run`, as proof that a live branch in a run-scope report is the parent pair's; that condition is reached only after the early return a few lines above it has already skipped every parent-pair target, and the parent's branch would belong to the parent's entry in any case, so the evidence does not hold.

### [MEDIUM] End's identity checks lack rows for the group check and for a reused record pid in the SIGKILL phase

**Location:** `_lyx/plan/02-runproc-owner-record.md:74`.
**Issue:** The Intent makes every signal follow a fresh start-time comparison and makes a group signal conditional on a live process still reporting the group id with a start time at or after the record's, in both phases.
`TestEnd` has rows for the happy group signal, a differing start time at the SIGTERM for a record, and descendant reuse at both phases.
No row reaches the group condition failing (no live member, or only members that started before the record) at either phase, and none reaches a recorded non-leader whose pid is reused between the SIGTERM and the SIGKILL.
These are the checks that keep `End` from signalling an unrelated process group, so each needs its row.

### [LOW] runLockHolderPasses has an input its rule never reads, and the paused pass is silent on an unknown holder

**Location:** `_lyx/plan/10-teardown-skip-readings.md:7`, `:56-60` and `:71`.
**Issue:** The signature carries `registered []runproc.Record`, and the card Uses `runproc.List` and `runproc.Record` for it, yet both pass cases compare only the marker pid with the holder pid, or read `paused` and `parentLocked`.
The one row that mentions "another registered pid of the slug" passes or refuses on the pid comparison alone.
Separately, the Intent says the holder passes in two cases but does not say whether the paused case applies when the holder is unknown, and the table has no such row.
The unknown holder with a covering pause and no parent lock record is the state an older-binary driver produces.

### [LOW] An unpaused run-lock refusal names a retry that cannot succeed

**Location:** `_lyx/plan/10-teardown-skip-readings.md:65`.
**Issue:** The skip reads true for an `ended` or `owned_elsewhere` verdict with no pause covering the slug, and nothing in the removal pauses such a run.
If its driver chain is still live on this machine, a Go driver holds the run lock for the run's whole life, so the refusal's step "retry once that pid ends" never succeeds.
`lyx tender pause <slug>` followed by the removal does succeed, because a pause with no parent lock record passes the holder.
The card already names that step in `busyError`'s way forward, so the two refusals of one situation disagree.

## Rejected

- Gated run with an output file missing is held at once while an ungated run waits for the bound: this matches D11, which evaluates a gated arrival "as at a plain turn end" and gives only the ungated run the expiry.
- The pass requiring the waiting marker to name the holder's own pid, narrower than D5's wording "or a process registered under the slug": D5's own example, a step merging under the parent lock, must refuse, and a registered push child waiting at the gate would pass it under the wider reading.
- Card 11's gone-worktree path judging `ended` and `owned_elsewhere` from the local Bolt: `originTargets` refuses a slug with no pair worktree, so no run-scope reconcile can run there, and D5 already lets the local Bolt decide when origin cannot be read.
- The skip-rule's hub-pause pass of a parent-lock holder that waits at the gate (the table row "marker naming the holder's pid passes, with a parent lock record"): D5 defines a holder at the network gate as the first pass case, ahead of "any other holder".
