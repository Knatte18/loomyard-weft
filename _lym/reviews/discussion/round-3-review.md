---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: "_lym/discussion/decision-record.md D1 act rule (lines 47-57), D4 (lines 169-172), D6 step 4, D9, Constraints PATTERN-owner-machine"
    summary: "A Finalize past its parent-side merge cannot be finished by the adopter: the merge commit sits on the old owner's local parent branch and the record never says it reaches origin or how the adopter tells the run is past the merge"
    fix: "In D4 and D1 state what makes a Finalize past its parent-side merge on a machine other than the one that merged, and what the adopter does. Pick one and write it into D2, D4 and the act rule. Either (a) the Finalize pushes the parent branch as part of the parent-side merge transition, before the board done write and before the transition's records push, so origin's parent branch holds the task's work whenever origin's records say past-merge, and the adopter derives past-merge from origin's fetched parent branch holding the task head; or (b) the adopter treats the Finalize as before its merge, re-runs the merge (which reports already up to date when the fetched parent holds the work), and the old owner's hand-over archives its local parent branch tip under the archive tag and resets that branch to its upstream so a later landing on that machine does not halt at ahead of upstream. In both cases define landing past its parent-side merge in D1 from tracked or fetched state, not from the local parent branch, and add an acceptance criterion where the old owner merged locally and was taken over before pushing the parent branch."
  - id: F2
    severity: BLOCKING
    class: consistency
    location: "_lym/discussion/decision-record.md D7 upgrade bullet and Bound, D8 release paragraph and Bound, Constraints (refusal-way-forward and shell-guard rows), Acceptance criteria D8 release lines, versus the arbiter entry at lines 449-453"
    summary: "The arbiter's removal of lym adopt --release is not folded into the body: D7, D8, Constraints and the acceptance criteria still specify the release verb, and the replacement flag for a stopped adopter's hold is not written anywhere"
    fix: "Fold the arbiter entry into the main decisions. Remove --release from D7 (upgrade bullet, Bound), D8 (the stop-on-release paragraph, the Kept and Retired lines if they mention it, the Bound), the Constraints rows for refusal-way-forward and shell-guard (keep the shell-guard row, retarget it to the new flag), and the three acceptance lines D8 release, D8 stopped adopter cleared by release, D8 stopped by a release after step 3. Replace them with D8 text for the takeover flag the arbiter asked the writer to name: an operator lym adopt of the same subjects on another machine with that flag rewrites the standing hold to name this machine, then runs its steps and check; the hold clears only at that adopt's green check or the fix task's green re-check. State its bound (it skips no check and lets no run go on before a green check), add the hold's subjects to D7's hold fields so the flag can verify same subjects, state that the stopped adopter stops at its next hold re-read, and add acceptance lines for the flag, its shell-guard deny, and the stopped adopter. Restate the D8 rationale and the PATTERN-dobby-runs text consistently (the PATTERN changes entry already has the green-check wording)."
  - id: F3
    severity: BLOCKING
    class: consistency
    location: "_lym/discussion/decision-record.md D9 last bullet, D8 step 6, D10 whole (lines 371-415), Scope Out, Constraints (fabric-boundary, shuttle-provider-seam, child-processes, code-conventions rows), Auto-mode assumptions, Acceptance criteria D10, versus the parent entry at lines 455-459"
    summary: "The parent's decision to drop the orch transcript mirror and every cross-machine restore is not folded in: D10 still specifies a mirror to the public weft repository, a tracked session record, a restore at start, and eight acceptance lines"
    fix: "Rewrite D10 to the parent's decision and delete the rest. Keep only the same-machine fix of D9 (end this worktree's own registered orch holder, resume the same session id, refuse over an unregistered holder). Remove the mirror, its ref, chunking, lease, the restore, the Facts block about the public repository and the 206 MB transcript, the Bound text on late mirrors, and the Rationale. State whether the orch session id stays in the machine-local orchengine state (the parent's decision implies it does, since no other machine resumes it) and drop the tracked _lym/orch record unless the same-machine fix needs it. In D9 change the last bullet so an orch start on a machine with no local transcript launches fresh with the carry-over, and drop the (D10) pointer. Remove D8 step 6's transcript rebuild, the Scope Out pointer to D10, the Constraints clauses that name the mirror or transcript ref, the mirror interval and chunk size from the code-conventions row and from Auto-mode assumptions, and every D10 acceptance line except one for the same-machine resume. Move the dropped mirror rejected alternatives in the support log to a single line saying the mirror was dropped by the parent."
  - id: F4
    severity: MEDIUM
    class: consistency
    location: "_lym/discussion/decision-record.md Open risks (lines 491-499)"
    summary: "All four Open risks bullets ask the parent for a call that an Added after Discussion entry already settled"
    fix: "Delete all four bullets. The public weft transcript bullet is settled by the parent entry on the mirror (F12), the offline bullet by the arbiter entry on no run acting while origin is unreachable (F15), the --release bullet by the arbiter entry on no release verb (F13), and the orch up during a hold bullet by the parent entry on the orch being up (F14). Leave the section with the single line none if the section must exist."
  - id: F5
    severity: MEDIUM
    class: design
    location: "_lym/discussion/decision-record.md D2 (lines 67-72, 84) and D4 (lines 173-177)"
    summary: "The records-before-code push order is worded for shed transitions only, but PushPairAnchored sends the code side first for every caller, so an old owner's commit-records or landing push after a takeover still moves origin's code branch and D4's claim that it moves only to a pushed transition's tip fails"
    fix: "In D2 state that the order change lives in fabricengine.PushPairAnchored itself and so holds for every caller of a pair push (the transition push, lym loom commit-records, Finalize's task push, the fabric push verb and the beam push), with the code side sent only after the records side landed and skipped when the records side is rejected. Add to D4 that a rejected records push over a takeover entry hands over for any pair push, not only a transition's, so no caller sends the code side after the takeover. Add one acceptance line where the old owner calls commit-records after the takeover and origin's code branch stays unchanged. Today pushanchored.go pushes the code side before the records side in its step function."
  - id: F6
    severity: MEDIUM
    class: design
    location: "_lym/discussion/decision-record.md D3 cancel bullet (lines 108-118), D1 queued state (line 44 and state 9)"
    summary: "A remove-cancel leaves its cancelled batten record after the cancel's own pair removal, because the record is said to go only with D11's teardown or lym fabric remove, so a slug released again reads cancelled and never queued and its dobby never starts it"
    fix: "In D3 name the record's remover for every path that ends a cancelled run's pair, the cancel's own removal included. State that the batten cancel with remove, D11's dobby teardown and lym fabric remove each delete the slug's cancelled record in the same step as the pair removal, and that lym batten cleanup is only the repair for a record left after a crash. Add to D1 that a released task with a cancelled record whose pair is gone reads queued, so an interrupted removal never blocks a re-release. Add an acceptance line where a cancel with remove is followed by a release of the same slug and the dobby starts the run."
  - id: F7
    severity: LOW
    class: design
    location: "_lym/discussion/decision-record.md D2 (lines 80-83), D7 upgrade bullet (lines 273-283)"
    summary: "The synchronous push holds a process-wide network admission through a wait on a held teardown lock with no bound, and an upgrade hold is written even when every standing adopt entry was dropped and no subject remains to re-run"
    fix: "In D2 bound the admission: release it and turn the push owed when the wait on a held teardown lock passes the existing push lock wait, so a pause reaches the process again. In D7 write the upgrade Bolt adopt hold only when at least one standing entry became a run pause intent, so the notice always names subjects to re-run; entries that were all dropped write no hold."
---

### [BLOCKING] Finalize past the parent-side merge is not finishable by the adopter

**Location:** `decision-record.md` D1 act rule, D4, D6 step 4, D9, Constraints.
**Issue:** D1, D4, D6, D9 and the `PATTERN-owner-machine` constraint all say the same thing: the owner finishes a landing past its parent-side merge, and after a takeover that is the adopter, once.
The wording is consistent.
The premise under it is not backed.
In `internal/landingshed/finalize.go` the Finalize merges the task branch into the parent pair's local branch, marks the board entry done ahead of the push (`markTaskDone`, line 708), and pushes the parent branch afterwards (`pushParent`, line 786).
`internal/fabricengine/replay.go` decides past-merge by reading that local parent branch (`finalizeEntered`, `ParentHoldsTaskWork`).
D4 says the parent-side merge "is a transition, on origin before the Finalize goes on", but a transition is a records commit.
The merge commit is on the old owner's machine until its parent push lands.
Two failures follow.
The adopter's local parent branch lacks the commit, so it reads the run as not past the merge, and D7's rule that a landing past the merge finishes under a hold does not apply to it.
The board `done` is pushed synchronously (D2) and reads `landed` (D1), a terminal state, while the parent branch on origin still lacks the work, and D11 tears the pair down.
The old owner, once handed over, keeps a local parent branch ahead of its upstream with a commit no one pushes, and a later landing on that machine halts at `syncParent`'s "ahead of its upstream".
The focus question has the answer: one machine finishes, but only when the merge commit is on origin, and the record does not make it so.

### [BLOCKING] The arbiter's removal of `--release` is not folded into the body

**Location:** `decision-record.md` D7, D8, Constraints, Acceptance criteria, against lines 449-453.
**Issue:** The arbiter entry tells the writer to drop `--release` from D7, D8, the Constraints, the declared `PATTERN-dobby-runs` change and the acceptance lines, and to state a replacement for a hold whose adopter stopped.
The PATTERN changes entry carries the new wording.
The body does not: D7's upgrade bullet and Bound, D8's release paragraph and Bound, both Constraints rows, and three acceptance lines still specify the verb the operator's decision rules out.
The replacement flag the arbiter asked for is nowhere.
A hold also names no subjects (D7 lists kind, machine, time), so the "same subjects" check the arbiter's flag needs has nothing to read.
A plan writer reading the main decisions builds the bypass.

### [BLOCKING] The dropped orch transcript mirror is still specified

**Location:** `decision-record.md` D9, D10, Scope, Constraints, Auto-mode assumptions, Acceptance criteria, against lines 455-459.
**Issue:** The parent's decision drops the mirror and every cross-machine restore, and keeps only the same-machine resume fix.
D10 still specifies a mirror of the transcript to the public `Knatte18/loomyard-weft` repository, a tracked session record, a restore at start, a lease protocol, and a Facts block.
The Constraints, Auto-mode assumptions and eight D10 acceptance lines depend on it.
Folding is the writer's job, and the plan writer never reads the Added entries as overrides of five sections.
The record also leaves open whether the session id record stays machine-local, which the parent's decision implies.
The round 2 deferral of the cross-machine resume research (its F11) is moot once the restore is dropped, and no research is owed.

### [MEDIUM] Open risks restate settled calls

**Location:** `decision-record.md` Open risks.
**Issue:** Each bullet asks the parent to confirm a call an Added entry already made.
The rubric makes this a consistency finding whose fix drops the line.

### [MEDIUM] The push order change is worded for transitions, the code order is shared

**Location:** `decision-record.md` D2 and D4.
**Issue:** `PushPairAnchored` in `internal/fabricengine/pushanchored.go` runs the code push, then the records push, in one step function, and its callers include `lym loom commit-records` (`internal/loomcli/wiring.go`), Finalize's task push (`internal/loomcli/landingdeps.go`), the fabric push verb and the beam push.
D2 states the new order for "a shed transition" and keeps `commit-records` "its synchronous push".
D4 then claims origin's code branch "moves only to a tip a pushed transition names".
For the transition push the claim holds: the rejected records side stops the code side, so the old owner's first transition after a takeover leaves origin's code branch unchanged.
For any other caller of the shared push, the old order still sends code first.
The adopter's own code push is bounded by the act rule's wait on the named code tip, which is adequate once the shared push is fixed.

### [MEDIUM] A remove-cancel's record outlives its pair and blocks a re-release

**Location:** `decision-record.md` D3, D1.
**Issue:** D3 says a cancel keeps the batten record and finishes it as `cancelled`, and that the record goes "with the pair's teardown, D11's or `lym fabric remove <slug>`".
A `lym batten cancel --remove` removes the pair through its own phase (`removePairAndBranch` in `internal/battencli/cancel.go`), which is neither of those.
The record then stays until someone runs `lym batten cleanup`.
D1 reads `queued` only with no batten record, and `cancelled` takes precedence, so a released slug is `cancelled` and no dobby starts it.
On the focus question: the done-run clear (`clearDoneRun` in `internal/battencli/cleanup.go`) judges only state `done`, so it never deletes a cancelled run's task branch, and `batten cleanup` refuses a kept cancel naming a way forward, so neither refuses forever.
The gap is the remove path.

### [LOW] Admission is unbounded across a teardown-lock wait, and the upgrade hold can name no subject

**Location:** `decision-record.md` D2 and D7.
**Issue:** `pauserecord.Admit` passes every network call of the process while the admission stands (`internal/pauserecord/hold.go`).
D2 holds it from before the commit until every side landed or the push turns owed, and a push that meets a held teardown lock "waits for its release", so a pause written during that wait reaches nothing in the process.
The focus question on pauses has this answer: a pause written between commit and push does not leave the first attempt unpushed, as D2 intends, and the owed retries wait at the gate as D2 says.
Only the unbounded wait is open.
Separately, D7 writes an upgrade hold for any standing entry, including entries dropped as landed or owned elsewhere, and the notice then names no subject to re-run.

## Prior rounds

Read after this review was saved.
The round 2 fixes for its F2 to F10 and F16 to F20 are present in the record and did not regress.
Round 2's fixer disputed nothing, so no finding here re-raises a disputed item.
Round 2's F1 fix (the adopter finishes the landing) stands as written; the first finding above questions the premise it rests on, which round 2 took as readable "after a fetch".
Round 2's F12 to F15 are answered by the Added entries, and the folding gap is the second, third and fourth findings.

## Focus verdicts

- Landing past the merge: D1, D4, D6 step 4, D8 step 3, D9 and the constraint agree that the owner finishes; the premise fails, see the first finding.
- Pair push order: the transition push is safe; other callers of the shared push are not, see the push order finding.
- Network gate and pauses: settled except the unbounded admission, see the last finding.
- Takeover race and local branches: D4's last bullet re-reads the hold and the owners from a fetched Bolt before rewriting the entry, and D8 step 3 fast-forwards the local branches to origin's tips after the takeover lands. No defect.
- Orch mirror: the parent dropped the mirror; the open item is the unfolded body, see the mirror finding.
- Cancelled and landed batten records: see the cancel finding. A landed slug whose record went reads `landed` from board `done`, as D1 says.
- Upgrade and teardown routes: `pairteardown.Request.Local` (`RemoveLocal`) removes a local pair with no origin step and no board write, so D6 step 4 names an existing route. The upgrade notice is covered by the last finding.
