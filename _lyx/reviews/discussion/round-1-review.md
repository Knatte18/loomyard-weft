---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D8 (first bullet; What it can skip or let through line)
    summary: D8 escalates before the judgment refusal, so a parent accept passes a CONVERGED round the refusal would re-judge (a BLOCKING review finding, a dropped ledger key), the brief shows only the decision findings, and the stated bound of nothing is false
    fix: In D8, keep the escalation ahead of the refusal for routing, but state that an accept on a decision escalation still applies the judgment refusal before acceptRound settles (a refused judgment re-judges instead of passing), and have the brief list every BLOCKING review finding and any refusal cause beside the decision findings; replace the What it can skip or let through line with that bound, and add an acceptance criterion - a decision escalation over a CONVERGED round whose review also holds a BLOCKING design finding is re-judged, not passed, on accept.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D8 (second bullet, never a settled generation at Call entry)
    summary: The never-a-settled-generation rule has no exit once the round's accept is settled, so a later entry (crash after settle, a commit-seam failure and resume, or an amendment needing a new generation) hits settleUnconverged's already-settled degrade, whose lyx loom start way forward re-enters the same degrade and loops
    fix: In D8's second bullet, limit the rule to a decision escalation whose decision file is absent or not yet settled, so a settled accept is a settled generation again and takes the existing settled-entry harvest, clear or new-generation path; add an acceptance criterion that a re-entry after a settled decision accept re-settles or starts a new generation over a changed subject, never degrades.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 (Each fork has 6 tool calls) and D6 (goal-scope reads; Both lenses decision-note rule)
    summary: D6's reads do not fit the fork's 6-call cap - goal-scope reads every Decision note with no bound and without the lyx board list that finds them, and board-fit spends all six calls before it can read a note the Both-lenses rule makes it honour, so it raises already-answered decision findings that D8 then escalates
    fix: In D6, give each lens a read set that fits six calls including the decision notes - for example goal-scope reads lyx board list once and then the task entry and up to four Decision notes, nearest first, and board-fit counts its decision-note reads inside its four reads (or names the notes from the lyx board list titles it already has) - and let each lens's list name the notes it read and any it could not reach; correct D5's fit sentence to match.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D11 (in-flight carry-across paragraph)
    summary: Merge-In's fixer attempt record is named after the row (FixerAttemptsPath, row-fixer-attempts.json) and RenameProducers rewrites only the status file, so a run carried onto Parent-Align reads a zero record - a fresh fix budget, no Baseline, no AlignFrom, no over-cap Halt - and D11 gives a fallback only for the fixer report
    fix: In D11, add that the Parent-Align row reads the old Merge-In-named fixer attempt record when its own is absent (and removes it where it removes its own), as it already does for the fixer report, and extend the acceptance criterion on carried-across runs - a run halted spent or over-cap on Merge-In resumes on Parent-Align with its spent attempts, baseline, AlignFrom and halt intact.
  - id: F5
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lyx/discussion/decision-record.md D1 and Open risks first bullet
    summary: The operator decided to keep the alignment agent where it approves the landing; D1 removes it whole because PR-Gate is gone, and only an Open risks line asks the parent whether a landing-time approval should survive, for example in the Parent-Align fixer - a question only the parent can settle
    fix: Put the question to the run's parent - does any landing-time approval replace the alignment agent's PR-Gate point, or does it go whole as D1 says - and record the answer in decision-record.md as a Decision (amending D1 if the parent keeps an approval), then drop the Open risks line.
  - id: F6
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D8 (arbiterstop bullet; hard-cap bullet)
    summary: A continue on a decision escalation whose round is at or past the hard cap at settle time is retired and re-escalated with cause hard-cap, which arbiterstop admits, so the arbiter can accept a round whose decision finding the parent never answered
    fix: In D8, state that a round escalated with cause decision keeps that cause when the hard-cap path re-escalates it (or that the arbiter refusal keys on the round's decision findings, not only on the cause), and add an acceptance criterion that such a re-escalation is still refused to the arbiter.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D8 (last bullet) and Constraints (PATTERN-shed-recipe-registry)
    summary: D8 names one row key, on the Discussion-Bouncer, but never says how Discussion-Burler-Fix learns to post no notice, so the two rows can disagree and a fix row silenced with the Bouncer key unset sends the parent nothing
    fix: In D8's last bullet, name how the fix row is switched - its own row key set together with the Bouncer's on loom's Discussion rows, or the fix row reading the Bouncer's key - and extend the Constraint line to cover it.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D10 (first bullet; With no parent bullet; What bounds it)
    summary: The darn writer asks for a board note and ends its turn, but a note wakes nothing, so a parent that only writes the note leaves the writer idle to writer_timeout_min; and with no parent, a turn end without output is held silently to the same timeout, so the stop and report reach nobody
    fix: In D10, have the writer ask the parent to write the note and then reply by SendMessage to wake it, and on any message re-read the board for the note; for the no-parent case, say the writer writes the change description naming the outdated entry as its stop report (or otherwise ends the run promptly) instead of ending its turn without output.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5
    summary: D5 names only review-discussion and twill-discussion, but the rewritten goal-scope also runs in the dormant discussion fan among emphasis lenses, and a hub lenses.yaml that tuned either lens keeps its old emphasis text inside the production fan
    fix: In D5, state the outcome for the discussion fan (it runs the rewritten procedure text, or drops goal-scope) and for a hub-tuned goal-scope or board-fit (reconcile reports it, or the hub keeps its text knowingly), and add the matching acceptance line.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D8 (accept bullet) against D7
    summary: An accept carries every decision finding into Open risks whatever the ledger says, so a finding the parent already answered with lyx loom decision add reaches Plan-Write as an open question beside the Added after Discussion entry that answers it, the answered line D7 calls a defect
    fix: In D8's accept bullet, carry a decision finding into Open risks only when no Added after Discussion entry recorded after the escalation answers it, or carry it with a pointer to that entry; adjust the matching acceptance criterion.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 (board-fit parent questions)
    summary: board-fit makes a landed parent commit that already makes one of the record's decisions a parent question in every case, while D6's last bound and D7 make a writer-settleable misalignment an ordinary fixer finding, so the same case routes two ways
    fix: In D6, narrow that board-fit bullet to a landed commit that contradicts a decision, or one that already makes it in a way the writer cannot settle by citing or dropping the decision; a commit that merely already makes a decision is an ordinary finding.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D8 (What bounds it line)
    summary: D8 names D7's class definition as its bound, but that bound is reviewer-prompt text only, and D8 fires at any severity whatever the judge ruled, so a writer's choice mis-tagged decision stops the run with nothing to narrow it
    fix: In D8's bounds line, state that D7's bound is reviewer discipline enforced by the rubric only, or let the judge's ledger close a mis-classed decision finding so that only a decision finding open in the ledger escalates.
---

The round judged the decision record and support log against the Discussion-Review rubric, the writer stencil and spec, and the board entry.
Each kept finding was checked against the code the record cites.
F4's row-named attempt record was confirmed at `internal/landingshed/fixerattempts.go` (`FixerAttemptsPath`).

### [BLOCKING] D8's accept bypasses the judgment refusal it now runs ahead of

**Location:** `_lyx/discussion/decision-record.md` D8, first bullet and the "What it can skip or let through: nothing" line.
**Issue:** `judgmentRefusal` refuses a CONVERGED judgment over a BLOCKING review finding, an open BLOCKING ledger entry or a dropped ledger key, and `rejudge` re-judges it (`internal/shedadapters/bouncer.go`, `judgmentRefusal`, `rejudge`).
D8 escalates before that refusal.
On the re-call, `standsAsRecorded` routes the round straight to `settle`, and an accept reaches `acceptRound`, which settles, approves and commits.
A round whose review holds a BLOCKING `design` finding that the judge wrongly closed therefore passes on the parent's accept.
The brief lists only the `decision` findings, so the parent never sees the BLOCKING one.
The carry-over takes only open ledger entries, so the BLOCKING finding is not carried either.
D8 states its bound as "nothing", which is false.
Under the rubric's attack-surface rule, that is a bypassable guard with a wrong stated bound.
All four lenses or the handler reached this; it is merged here.

### [MEDIUM] The never-a-settled-generation rule never ends

**Location:** D8, second bullet.
**Issue:** `settledGeneration` today counts a settled accept as settled (`bouncer.go:833-843`), which gives a later entry the harvest, clear and new-generation paths.
D8 takes every escalated round off that path with no exit once its accept settles.
A re-entry then reaches `settleUnconverged`'s `case settled`, which degrades and names `lyx loom start` as the way forward.
That command re-enters the same branch, so the run loops.
An amendment over the settled record never starts a new generation either.
The interactions, walkthrough and existing-paths lenses found this independently.

### [MEDIUM] The lenses' reads do not fit the fork's six calls

**Location:** D5's fit sentence and D6's reads.
**Issue:** `burler-cluster-fan.md` caps a fork at six tool calls.
`goal-scope` reads every `Decision for <slug>:` note, with no bound.
None of its reads lists the notes, and each note costs one `lyx board get`.
`board-fit` spends exactly six calls on list, peers and four reads, yet the Both-lenses rule makes a note win over the entry.
A fork that cannot read the note raises a `decision` the parent already answered, and D8 stops the run on it, the outcome D7 was written to prevent.
The handler and three lenses raised it.

### [MEDIUM] D11's carry-across loses the Merge-In fixer attempt record

**Location:** D11, in-flight paragraph.
**Issue:** `FixerAttemptsPath(scratchDir, row)` names the record `<row>-fixer-attempts.json`, and the record exists "so a resume never grants a fresh cap".
`RenameProducers` rewrites only status fields (`internal/shedengine/doc.go`, "Row renames").
After the rename, `ReadFixerAttempts` finds no file and returns a zero record: a fresh budget, no `Baseline` for the cap judgment, no `AlignFrom`, and no over-cap `Halt`.
D11 adds the old-name fallback only for the fixer report.
Three lenses raised this.

### [MEDIUM] Whether a landing-time approval survives is the parent's call

**Location:** D1 and the first Open risks bullet.
**Issue:** The board entry records the operator's decision: "keep it at `pr-gate`, where it approves the landing".
The record correctly finds that PR-Gate is gone (`fc75aef79`).
It then picks for the operator, dropping every landing-time approval, and leaves only an Open risks line asking whether the parent wants one kept.
That question goes back on an explicit operator decision.
Only the parent can settle it, so it is filed as `decision` and goes to the parent, not the fixer.

### [LOW] A hard-cap re-escalation strips the decision cause

**Location:** D8, the arbiterstop and hard-cap bullets.
**Issue:** `retireContinue` followed by `escalate(round, EscalationHardCap)` replaces a `decision` record with a `hard-cap` one.
`arbiterstop.Eligibility` admits `hard-cap` (`internal/arbiterstop/arbiterstop.go:77`).

### [LOW] The fix row's silence has no stated switch

**Location:** D8, last bullet, and the Constraints.
**Issue:** D8 names only the Bouncer's key, so the two rows can be configured apart.
Then the parent gets either no message or two.

### [LOW] D10's wait is woken by nothing, and the no-parent stop is silent

**Location:** D10.
**Issue:** A board note sends no message to the held writer.
Shuttle holds a turn end without outputs until `writer_timeout_min` (180 minutes by default, `internal/darnengine/template.yaml`), and with no parent there is no hold notice.
This merges bound-breaker F4 and walkthrough W6.

### [LOW] Other fans and hub layers that run the rewritten lens texts

**Location:** D5.
**Issue:** The rewritten `goal-scope` also runs in the `discussion` fan (`internal/lenses/template.yaml`), alongside emphasis lenses.
D5 also leaves out a hub `lenses.yaml` that defines either lens: its own text wins.
Raised by the existing-paths lens and the handler.

### [LOW] An accept carries findings the parent already answered

**Location:** D8's accept bullet.
**Issue:** The carry-over runs "whatever the judge's ledger says".
So it writes a finding that a later `Added after Discussion` entry answers into Open risks, and no Discussion round is left to drop it.

### [LOW] An already-made decision in a landed commit routes two ways

**Location:** D6, the `board-fit` bullet.
**Issue:** D6 sends a landed commit that already makes a decision to the parent every time.
D6's last bound and D7 send a misalignment the writer could settle to the fixer.

### [LOW] D7's bound is prompt text only

**Location:** D8's bounds line.
**Issue:** D8 fires at any severity "whatever the judge ruled".
So nothing past the reviewer's own classing keeps a writer's choice from stopping the run.

## Focus departures

None.
On the focus points that raised no finding:

- No Decision edits this task's own board entry or files or promotes a task.
  The other-entry rewrites in Scope Out are the orch's work, outside the run, so that point is not a defect.
- The `loom-beam-darns` Open risks line concerns a future note's design, not a fact this design rests on, so it is no deferred research question.
- D2's in-flight claims held against the embedded recipe and the gate list.
- No requirement the plan writer needs lives only in `support-log.md`.
- D6's both-lenses bounds partly restate D7, but each carries a rule its own reader needs, so no relocation finding was raised.
