---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:127
    summary: "D7 answer-key test for a card that does not yet carry the entry never settles for a named card of a done batch, and names cards by a number that insert-card shifts"
    fix: "In D7's answer-key sentence (the one beginning 'The Plan-Burler-Review answer-key instruction gains') replace 'a finding for each card that entry names and that does not yet carry it' with a definition of carried: an entry's change is carried when every card the entry names carries it, or, for a named card whose batch is done, when a card inserted with `lyx loom insert-card` after the last begun batch carries it; the instruction raises a finding for a named card only while its change is uncarried in that sense. Add in the same sentence that a decision names each card by slug as well as number, because `lyx loom insert-card` renumbers every later card, and carry the slug rule into D8's sentence that both stencils word the decision as a concrete change to named cards. Add one acceptance criterion: an amending entry naming a done card is satisfied by an inserted card that carries the change, and the next round raises no finding for the done card."
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:169
    summary: "D9 step 3 matches a not_found location only to cards whose targets hold the file, but a card that edits a member glyph names no file, so the common card shape traces to nothing"
    fix: "In D9 step 3, state how a card whose target is a member glyph matches a `not_found` location: through `internal/planglyph`, which owns the quarry repo handle, resolve each member-glyph target of the live cards at the record's `Head`, take the file each resolves into, and match a `not_found` location to every live card whose targets hold that file, its file self glyph or a member resolving into it; a member target that does not resolve at `Head` falls back to the file and unit rule as written. Add an acceptance criterion beside the unused-import one: an unused-import build error in a file where a card targets only a member glyph names that card. If the writer chooses not to resolve member targets, instead add this limit to Open risks and narrow the unused-import acceptance criterion to a card that targets the file."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:115
    summary: "D7 decides that a failed commit landed from the amendment record alone, but every loom status commit stages _lyx/loom, so the amendment can land without the decision entry"
    fix: "In D7's commit-failure paragraph, make the landed test read both files at HEAD: after a commit error the verb reads the amendment record and the decision record as committed, the latter as `committedHoldsEntry` does for the entry heading, and reports success only when the amendment equals this call's bytes and the decision record holds the entry's heading. When HEAD holds the amendment bytes but not the heading, keep both files and refuse with `lyx fabric commit` as the way forward, as for an unreadable HEAD. Soften the sentence that the decision entry and the amendment record land in one commit to say they do unless a status commit, which stages the loom durable directory, lands the amendment record first. Add the partial-landing row to the acceptance criterion on the failed second call."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:114
    summary: "D7 restores the amendment record only when the append fails, and says nothing for the check-findings refusal that the verb reaches with a nil error"
    fix: "In D7's sentence beginning 'When the append fails', reword the condition to 'When the append returns an error or check findings', and state the order the verb uses: the amendment record is written before the append, so a findings refusal restores it to its bytes before the call, removing it only when the call created it, exactly as an append error does. Cover the findings refusal in the acceptance criterion that a commit which did not land leaves the amendment record as before the call."
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:171
    summary: "D9 spends a range rule on failed and in-flight batches that cannot exist at Merge-In or Publish, and never names the field that holds a batch's recorded head"
    fix: "In D9 step 3, name the recorded head as the batch's `CardSHAs` element in Webster's state (`BatchState.CardSHAs`), which the done paths set and the failed path does not. Delete the clause that gives a failed or in-flight batch the range from its start SHA to the record's `Head`, since a Webster row that reached Done has every batch terminal done (`verifyEveryBatchDone`) before Merge-In or Publish runs; for a batch whose record holds no `CardSHAs` element, such as one written before the field, give the card an empty range. Rewrite the acceptance criterion 'A matched card of a batch with no recorded head takes the range from that batch's start SHA to the record's `Head`' to the empty-range case, and keep its second half, the later range after a done batch."
---

### [MEDIUM] Answer-key carry test is unstable for done cards and for renumbered cards

**Location:** `_lyx/discussion/decision-record.md:127`
**Issue:** D7 makes each amending entry "a finding for each card that entry names and that does not yet carry it", and lets the fixer resolve it either by editing an editable card or by inserting a card.
A card of a done batch is frozen (`done-card-edited`), so the only fix for an entry that names one is the inserted card, and that card is by construction not one the entry names.
Read literally, the named done card still "does not yet carry it" next round, so the finding recurs until the Plan-Review budget halts the route.
The no-card branch of the same sentence does say "until some card carries it", so the two branches disagree on whether an inserted card satisfies the entry.
Separately, `planparser.InsertCard` moves every card from the insert point up one place, and D8 has the parent, orch and arbiter word the decision as a change to named cards with no rule for how a card is named.
An entry written as card 7 before an insert points at a different card after it.
The writer's own D9 note that begun cards keep their ids confirms the later cards do not.

### [MEDIUM] `not_found` matching cannot reach cards that target members

**Location:** `_lyx/discussion/decision-record.md:169`
**Issue:** Quarry's `not_found` answer carries the file's self glyph in `Unit` (`quarry/internal/engine/enclose.go`, `answerFromOutcome`), and D9 matches it to "every live card whose targets hold that file or its unit".
Per the plan spec a card's `Edit` targets are existing symbols, and a member glyph spells a package unit and a member, never a file; only a file self glyph or a path names one.
So an unused import in `foo.go`, whose cause is a card that edited members of `foo.go` and changed its imports, matches no card that lists the member glyphs, and goes into `untraced`.
The acceptance criterion only claims the case "in a file a card targets", which hides the gap, and the Open risks do not list it.
Resolving a member glyph to its file at the record's `Head` is something `internal/planglyph` already does for other checks, so the choice is open rather than infeasible.

### [LOW] Landed-commit test trusts the amendment record alone

**Location:** `_lyx/discussion/decision-record.md:115`
**Issue:** After a commit error D7 reads only the amendment record at HEAD, and when it equals this call's bytes treats the commit as landed and keeps both files.
`statusCommitPathspec` (`internal/loomcli/wiring.go`) stages the whole loom durable directory, `_lyx/loom`, on every status commit, and `_lyx/discussion`, where the decision record lives, is staged only by `commitDiscussion`.
A status commit that runs between the amendment write and the verb's commit therefore lands the amendment record alone.
The verb then reports success while the decision entry the amendment's title names is uncommitted, and on adoption elsewhere the amendment record names an entry that is not there.
The "same commit" sentence is true of the verb's own commit and not of the file's whole life.

### [LOW] Findings refusal leaves the amendment record unrestored

**Location:** `_lyx/discussion/decision-record.md:114`
**Issue:** `AppendDecision` restores the decision record and returns findings with a nil error when the check flags the append, and `decisionVerb` then refuses.
D7 restores the amendment record "when the append fails" and restores the decision record "as it already does on a check finding", which reads the findings case as already handled and never says the amendment record goes back too.
A reader following it leaves a new title in the amendment record with no entry in the decision record, which the guard admits and the answer key cannot resolve.
D7 also never says the amendment record is written before the append, which the restore wording presumes.

### [LOW] Range rule covers batch states that are unreachable and names no head field

**Location:** `_lyx/discussion/decision-record.md:171`
**Issue:** The Webster row reaches Done only after `verifyEveryBatchDone` finds every batch terminal with status done, and Merge-In and Publish run after it, so at the trace no matched card belongs to a failed or in-flight batch.
D9 still writes a range rule for them and an acceptance criterion for "a batch with no recorded head".
The "recorded head" is never tied to a field: `BatchState.CardSHAs` is set on the done paths only, while a failed batch's head lives in `Digest.HeadSHA`, so the two readings of "has a recorded head" differ for the very batches the clause targets.
Relocating or deleting unreachable design text is a legitimate finding under the rubric, and the plan writer would otherwise build and test a branch no run reaches.
This reverses part of what round 2's range finding asked for, which asked that the range be defined for a failed or in-flight batch; the evidence above, `verifyEveryBatchDone` and the recipe's Webster, Merge-In, Publish order, is what that finding did not weigh.
The branch is reachable only by an operator goto that passes an unfinished Webster row, which is no case to design a trace range for.

## Rejected

- D7, record location: `_lyx/loom` is `LoomDurableDir`, tracked run content committed with the run, so the record is not only under `.lyx`; it reaches another machine through the fabric as the decision record does, and its pathspec joins the decision commit. Only the error path is weak, F3.
- D7, start of the judged round: `round-<N>-usage.yaml` holds the review half's `start` as a `time.Time` that yaml.v3 writes at nanosecond precision, with `omitempty` leaving a zero start absent, and a JSON `round_started_at` is also nanosecond, so the comparison is at equal precision as an instant; a round-0 skip-seam settle records none, and the skip seam answering false while the record exists means no round-0 settle can follow a record.
- D7, the bound on goto over a settled Plan-Review: it is stated for every case, including a settle with no known start, and it holds as stated; the stale-record window after an unknown start admits a goto that re-runs a review round raising nothing, a cost and not a bypass, so no finding.
- D7 and D8, finding for an unnamed card: as worded, an entry that names cards raises findings only for those cards, and an entry that names none raises one finding until some card carries it, so an unnamed card never draws a finding; F1 concerns the carry test, not this.
- D1 against D4, order of two cards fencing one entry ID: it holds, because D4 makes a pair of `Extends` on one ref an edge when both cards fence a line for one entry ID, and the acceptance criteria pin it; `tools/plandag`'s `-additive-guess` is a measurement flag and not the batch-order rule.
- D9, a test or member a card creates under a `plan:` handle: `planparser.RefMember` maps the handle to the glyph it becomes, and quarry's `found` answer names that member at `Head`, so the match works; the gap is limited to `not_found`, F2.
- D9, location outside every member that quarry answers `not_found`: answered through `Unit` for a card that targets the file, and into `untraced` otherwise; only the member-target case is open, F2.
