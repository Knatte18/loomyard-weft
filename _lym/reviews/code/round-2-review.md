---
verdict: BLOCKING
prior:
  - id: F1
    status: resolved
    answer: "scopeClosedRefusal now admits a bug entry's add titled No test seam or No bug fixed that amends no plan, changes no PATTERN entry and drops no Create target (onlyLiftsRedFirst plus the BugLabel check), with rows in TestDecisionCalls_RefusedOnceTheScopeCloses. darnStopWayForward, planConflictWayForward, createTargetWayForward, loomRerouteCapWayForward, parentReviewWayDecision and its help, and the Parent-Align escalate in fixerrun.go each name the board-note route, and each keeps a test asserting its message. The narrower exemption the fixer chose is sound: a title alone would let --amends-plan, a PATTERN change or --drops-create through."
  - id: F2
    status: resolved
    answer: shedadapters.SegmentRan counts a round review, a generation record, a settle record or an archived sibling the ArchivedRunDirSegment rule maps to the segment, and keeps the error for other read failures; archiveRunDir, used by both a new generation and the entry clear, always leaves that sibling, so the scope stays closed after either, including a generation that writes no round.
  - id: F3
    status: resolved
    answer: "Settled by the added entry No operator exemption at PR-Review: the code admits no caller at PR-Review, as scopeClosedRefusal already has it. The description and role-file wording that entry asks to change is not yet changed, and the new finding F14 carries that work."
  - id: F4
    status: resolved
    answer: Each named site now names the closed-scope route, in role-autonomy.md, role-landing-halts.md, orch-role.md, darn-template-write.md, sampler-template-agent.md and loom-rubric-code-review.md. Leaving the red-first ask in darn-template-write.md unchanged is right, since decision add takes that record in a closed scope.
  - id: F5
    status: resolved
    answer: TestScopeClosedAt drives scopeClosedAt over a t.TempDir() reviews directory, with rows for a missing directory, an empty run directory, a round review, both carried records, an archived sibling, another segment's archive, an unreadable run directory and arras.
  - id: F6
    status: resolved
    answer: TestDecisionSwitches_SetOnTheRowsThatAskTheParent pins scope_closed on the three Code-Bouncer rows, and the new TestBouncerEntry_BlockingEscalatesAndColorSegment row asserts the closed-scope reason, so dropping either the key or the ScopeClosed pass-through fails a test.
  - id: F7
    status: resolved
    answer: answerRoute takes the round and at the hard cap says only to file every call as a board note; the at-cap closed-scope subtest checks that no --in-scope or continue verb appears in the reason, notice or brief.
  - id: F8
    status: resolved
    answer: The prompt row for InScope, the closed-scope hard-cap subtest, the RecordInScopeContinue rows for a second decision, no round and an unparseable review, the malformed InScopeFindings test and TestRecordCircling cover each named branch.
  - id: F9
    status: resolved
    answer: shed-recipe-spec.md documents scope_closed, its default, its effect and the shipped rows that set it.
  - id: F10
    status: resolved
    answer: The add help again states that a failed board copy is record_store_error and never fails the verb, that --by and --amends-plan apply to every --from entry, that all entries land in one commit or none does, and that --drops-create lets Webster's done-check skip the target and is recorded as its own call.
  - id: F11
    status: resolved
    answer: The loom-plan-spec rule now opens with the same budget-line condition as the plan-cards block and the plan-review rubric.
  - id: F12
    status: resolved
    answer: "plan_tier0_budget_sec is CategoryGuard. PATTERN-config-layers names a key that switches off or hollows a review's check as Guard, and a value of 0 or a large value does exactly that to the plan review's budget check."
  - id: F13
    status: resolved
    answer: The merge-danger blast radius names loom, gate and the review perches, and adds the older-binary refusal of a gate.yaml carrying plan_tier0_budget_sec.
findings:
  - id: F14
    severity: BLOCKING
    class: design
    location: contracts/orchrole/orch-role.md:156
    summary: "orch-role.md tells the parent that the operator's own feedback on the pull request stays outside the closed scope, and the change description still says that feedback needs no record entry. The added entries No operator exemption at PR-Review and The parent's PR-Review ruling governs the description's wording rule the other way: that sentence is dropped, and operator feedback found after the scope closes becomes a board note like any other late call. The arbiter's clause that kept both wordings is withdrawn."
    fix: "In contracts/orchrole/orch-role.md:156, replace the sentence opening \"Two things stay outside the line:\" with one that keeps only Finalize outside the line and files operator feedback as a note, for example: \"Finalize's fixes to a merge or test failure stay outside the line, and their escalations still take `lym loom decision add`; feedback the operator gives after the scope closes, on the pull request or elsewhere, is a board note for a new task like any other late call.\" In the same commit, delete the sentence \"Feedback the operator gives on the pull request needs no entry in the record.\" from _lym/landing/summary.md's The scope line section (line 27), adding nothing in its place."
---

### [BLOCKING] Role file and change description still exempt operator feedback on the pull request from the closed scope

**Location:** `contracts/orchrole/orch-role.md:156`, and the same claim at `_lym/landing/summary.md:27`

**Issue:** The new orch-role paragraph ends: "Two things stay outside the line: the operator's own feedback on the pull request, and Finalize's fixes to a merge or test failure, whose escalations still take `lym loom decision add`."
That sentence repeats the unamended board entry.
The record's added entries amend it.
The parent's entry *No operator exemption at PR-Review* says operator feedback found after the scope closes becomes a board note, like any other late call, and that the change description drops its claim that such feedback needs no record entry.
The arbiter's entry *The parent's PR-Review ruling governs the description's wording* withdraws the clause that kept both the summary's sentence and orch-role.md's "the operator's own feedback on the pull request" wording.
It repeats the order to drop the sentence and to word late operator feedback as a board note.

Neither change has landed.
A parent that reads orch-role.md as it stands takes operator feedback on the pull request to be exempt from the line.
Yet `scopeClosedRefusal` refuses `--by operator` at PR-Review, as both entries require.
So the role file tells the parent about a route the closed scope refuses: focus question 3 asks about exactly this case.
The summary's line 27 is a claim the amended answer key orders removed.
This is the work F3 left behind once the parent ruled on it.

## Focus departures

None. Every focus question was reached:

- Ways forward once the scope is closed: `redfirstgate.go` names a `No test seam` record, which `decision add` takes on a bug entry in a closed scope. `darnwrite.go`, `webster.go`, `recordbatch.go`, `wiring.go`, `parentreview.go`, `fixerrun.go` and `bouncerdecision.go`, at the hard cap too, each name the board-note route. A test asserts each changed message. The one route the closed scope refuses is in orch-role.md, filed as F14.
- `scopeClosedAt` after an archive: `SegmentRan` reads the archived sibling and both carried records, and `TestScopeClosedAt` drives it over real directories. Not a defect.
- Role files and stencils: apart from F14, the darn, sampler and loom Code-Review stencils and the role files name only routes the closed scope accepts.
- `scope_closed` tests: dropping the key from a shipped row fails `TestDecisionSwitches_SetOnTheRowsThatAskTheParent`. Dropping the pass-through fails the closed-scope row of `TestBouncerEntry_BlockingEscalatesAndColorSegment`. Removing the key from `configRejectUnknown` makes every shipped recipe fail to parse. Not a defect.
- `plan_tier0_budget_sec` category: `CategoryGuard` is the category PATTERN-config-layers requires for a key that switches off or hollows a review's check. Not a defect.

## Rejected

- `onlyLiftsRedFirst` has no test row for a `--from` batch that mixes a red-first title with another title, nor for `--drops-create` with a red-first title. This is a coverage gap below BLOCKING, so an incremental round cannot raise it.
- Mutation, redundancy and reach reports: none is current, so no finding is raised about them.
