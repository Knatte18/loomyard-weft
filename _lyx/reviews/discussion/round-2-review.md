---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:70
    summary: A reroute-cap halt at the Webster row names the loom-wide parent-branch way forward, which does not apply to a plan conflict, and D3/D5 leave it unaddressed
    fix: In D3, add that the reroute-cap halt now reachable at the Webster row needs a way forward that fits a plan conflict (edit the named pending cards by hand so they match the tree, then lyx loom resume), and decide how it is supplied, since loomcli tells one shared clause, loomRerouteCapWayForward (run lyx loom resume once the parent branch has settled), to every row; either make that told clause row-neutral or let the Webster row's cap halt carry its own clause. In D5, add the matching refusal-spec loom row for a Webster reroute-cap halt, and add an Acceptance criteria bullet that a capped Webster reroute halts naming that way forward.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:61
    summary: The adapter writes the last-routed file before shed decides whether the reroute is taken, so after a reroute-cap halt the resumed Webster halts as a repeated conflict that never got a plan review
    fix: In D3, change the last-routed rule so a conflict counts as routed only once a plan review has seen it, for example by having Plan-Burler-Review (or the adapter on the next Webster entry, from shed history showing a reroute entry routed_to Plan-Burler-Review) write the last-routed file instead of the Webster adapter at verdict time; or, if the write stays in the adapter, state in D3's Bound and Open risks that a conflict whose reroute shed capped is recorded as routed, so resuming the cap halt halts with the survived-a-plan-review reason instead of granting the three further reroutes, and make the repeated-conflict halt's reason not claim a review took place. Add an Acceptance criteria bullet for whichever behavior is chosen.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:48
    summary: D2 claims a record exists only when the latest begin-batch refused with plan_conflict, but a begin-batch that refuses before the removal point keeps an earlier record
    fix: In D2, place the removal at the first step of the begin-batch verb, before plan parsing, batch-number parsing, lock acquisition, state load and batch derivation, and say so; or narrow the sentence on line 49 to begin-batch calls that reach the mutate lock and name the earlier refusals (bad batch number, plan parse failure, state load or batch derivation failure, no run in progress) as covered by D3's stated exposure.
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:63
    summary: The D3 bullet list has no blank line after it, so every following D3 line, Bound and Rationale included, renders as a continuation of the no-record bullet
    fix: Insert a blank line after line 63 (the no-record bullet), so the list ends there and the line A successful begin-batch removes the last-routed file too starts a new paragraph.
---

### [BLOCKING] Webster's reroute-cap halt names a way forward that does not apply

**Location:** `_lyx/discussion/decision-record.md:70`

**Issue:** D3 gives the Webster row its first `on_reroute` and relies on shed's reroute cap as the bound for distinct conflicts and for renumbered findings (line 70 and line 79).
That makes shed's reroute-cap halt reachable at the Webster row.
Its reason ends with the told `Shed.RerouteCapWayForward` (`internal/shedengine/reroute.go:76`), and loom tells one shared constant for every row: `loomRerouteCapWayForward` in `internal/loomcli/wiring.go:1005`, which reads "run \"lyx loom resume\" in the task worktree once the parent branch has settled".
That clause was written for the `Merge-In` reroutes of Publish, PR-Gate and Finalize; for a Webster plan conflict the parent branch is irrelevant, and resuming without a plan edit re-runs the same refusal.
`PATTERN-refusal-way-forward` requires every refusal reachable through loom's verbs to name its way forward and to carry a spec row; the decision record neither adjusts the clause nor adds a loom row for a capped Webster reroute (D5 lists only the reroute and the repeated-conflict halt).
The constraint binds, so this is BLOCKING.

### [MEDIUM] Last-routed is recorded before the reroute is known to be taken

**Location:** `_lyx/discussion/decision-record.md:61`

**Issue:** D3 has the Webster adapter write the findings as the last routed conflict and then return `Stuck` with `Reroute` set.
Shed decides afterwards whether to take the edge: at three reroutes since the row's latest Done, `rerouteStuck` halts blocked with `reroute_capped` instead (`internal/shedengine/reroute.go:73-79`), and the shed doc says resuming that halt grants three more reroutes.
Webster returns Done only when its whole run finishes, so three distinct conflicts in one Webster run reach the cap at the fourth.
On resume, shed re-calls the Webster producer, `Run` removes the conflict record, begin-batch refuses again with the same findings, and the adapter finds them equal to the last-routed file it wrote at the capped attempt.
It then halts with the reason that "the same conflict survived a plan review", though no review ever saw it, and the three reroutes the resume granted can never be used for that conflict.
D3's Bound ("one conflict gets one plan review") is therefore false at the cap, and neither the Bound nor Open risks states it.

### [LOW] D2 overclaims when a conflict record can exist

**Location:** `_lyx/discussion/decision-record.md:48`

**Issue:** D2 puts the removal "under the mutate lock and before its re-resolution" and concludes on line 49 that "a record therefore exists only when the latest begin-batch of this Webster call refused with `plan_conflict`".
`internal/webstercli/beginbatch.go` refuses before the lock on an invalid batch number and on a `planparser.ParsePlan` failure, and after the lock but before `websterengine.BeginBatch` on a state load failure, a missing run and an `executionBatches` failure.
None of those reaches a removal placed under the lock before re-resolution, so each leaves an earlier record in place, and a master that then goes stuck is rerouted.
D3's stated exposure covers the consequence, so the routing risk is bounded, but the sentence on line 49 is not true as written and the acceptance criteria pin only quarry-unavailable and fingerprint mismatch.

On the focus question itself: quarry-unavailable, fingerprint mismatch (`PlanEditError`, which runs before re-resolution inside `BeginBatch`) and success all leave no record provided the removal precedes `PlanEditError`, which "first" in D2 implies; D3's Bound states the remaining exposure of a master stuck for another reason after a `plan_conflict` refusal.

### [NIT] D3's bullet list swallows the rest of the decision

**Location:** `_lyx/discussion/decision-record.md:63`

**Issue:** Lines 61-63 are a bullet list followed on line 64 by "A successful begin-batch removes the last-routed file too." with no blank line.
Under CommonMark that line and every later line of D3, including `Bound:` and `Rationale:`, are lazy continuation lines of the "no record" bullet, so a rendered view shows the Bound and Rationale inside that bullet.

## Focus verdicts

- D2 removal and write across a Webster call: holds for calls that reach the removal point; see F3 for the earlier refusals. D3 states the exposure of a master stuck for another reason after a `plan_conflict` refusal, bounded by D4.
- D3 repeat comparison: states the compared key exactly (the set of `Finding.Error` strings, batch number and cause excluded, order ignored), so a reordered equal set is equal. `Finding.Error` renders `check/card[severity]: detail` (`internal/planindex/planindex.go:41`), and the delete-before-reference detail embeds card numbers and the tree's `file:line`; the tree does not move between the reroute and Webster's re-entry, so the Rationale's claim (one review while card ids and numbers stay unchanged, renumbering bounded by the cap) matches the comparison, except at the cap itself (F2).
- D3/D5 repeated-conflict halt: the Stuck reason and the spec row carry the same way forward (hand edit of the named pending cards, then `lyx loom resume`), and neither names a denied command; `lyx loom resume` re-calls the halted producer per `internal/shedengine/doc.go`. Not a defect. The way-forward gap is in the reroute-cap halt instead (F1).
- D3 hard cap on a rerouted round: D3 and Open risks state that the round continues as N+1 while shed's bounce count resets, and that at or past the hard cap a CONTINUE with no design-wrong entry auto-accepts back to Webster while a design-wrong one escalates with cause hard-cap. This matches `softCapRule` in `internal/shedadapters/bouncer.go`, where the design-wrong check precedes the hard-cap escalation. Not a defect.
