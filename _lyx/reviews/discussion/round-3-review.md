---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:39-40, 55-58
    summary: >-
      D1 counts a decision-record entry added with --by operator as a recorded operator decision,
      but lyx loom decision add documents --by as an unverified label that the parent's own fork can pass,
      and D1's Bound does not say so
    fix: >-
      In D1's definition, its Bound and the brief's restated definition, state that --by is an unverified label
      (internal/loomcli/decision.go restricts only --by arbiter), and bound the count:
      a --by operator entry counts only when it is already in the committed decision record before the escalation,
      and the fork never records with --by operator, only with --by parent as step 1 says.
      Add the same one-line rule to the brief's changes list under D1.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:39
    summary: >-
      D1 counts the task's board entry and an operator's board note wholesale,
      yet board bodies mix operator decisions with agent suggestions and operator views, and a note carries no author
    fix: >-
      In D1's definition and the brief's restated definition, narrow the board sources to text the entry itself marks
      as an operator decision with a date, as in the headings Operator decision (date) of the task entry.
      Say that an Idea or agent-suggestion section, and an operator view not marked as a decision, never count.
      Say that a board note counts only when its body states it records an operator decision.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:39, 48
    summary: >-
      D1's definition of a recorded operator decision names the support log's Interview,
      but the brief's materials list, as D1 amends it, names only the envelope and the existing four inputs
    fix: >-
      In D1's list of brief changes, add to the materials the support log in the decision record's directory,
      read when the definition needs its Interview section,
      or drop the support-log source from the definition if the fork is not meant to read it.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:60
    summary: >-
      The role bullet that tells the orch to decide a design call a child cannot make and record it with lyx loom decision add
      stays in the same section, against D1 and D2, which forbid the orch to decide a review escalation as a design call
    fix: >-
      In D1 or D2, name the bullet beginning A design call the child cannot make in the role section's Escalations from a child
      and reword it in the same change, limited to escalations that are not review escalations,
      or state that a review escalation never takes it.
      Add the reworded bullet to Scope's first line and to the Acceptance criterion on the rendered role file.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:45-55
    summary: >-
      The brief's decide fragments open with Record your decision on the segment, which frames accept or continue as the fork's own call,
      and neither D1's brief changes nor Scope's file list covers the two fragment files
    fix: >-
      In D1's list of brief changes, add that the decide fragments bouncer-fragment-decide-open.md and bouncer-fragment-decide-capped.md
      reword their lead-in to name the operator decision applied as the ground for the option,
      and add both files to Scope's In list.
      If the wording is meant to stay, say so in D1 with the reason.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:182-183, 237
    summary: >-
      D9 names the Merge-Review perch as the owner after every merge-in, but merge_max_bounces set to 0 makes the perch's Bouncer settle at once without a review,
      and D9 and the Acceptance criterion state no bound for that
    fix: >-
      In D9's first bullet, state that the check holds while the perch's budget is above 0 and say what covers a budget of 0,
      for example the orch's rule that a workaround is removed with its fix,
      or add the budget-0 case to Open risks.
      The rest of D9's claim holds as written, since the overlap seam compares changed files and CLAUDE.md is changed on both sides,
      so the perch reviews even a clean merge-in.
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:184-186
    summary: >-
      When this run lands first, D9's board note has no named reader,
      since the reed run is past Discussion and Plan and its perch measures against its own record, not the board
    fix: >-
      In D9's second bullet, name who acts on the note when reed-add-parentless-placement lands,
      for example the orch on that run's landed notice, handing a fresh agent the removal of the ORCHESTRATOR.md entry as the stencil's rule that a workaround is removed with its fix directs,
      or say in D9 that the note is only a pointer and give that case to Open risks.
---

### [MEDIUM] D1's operator-decision test rests on an unverified label

**Location:** `_lyx/discussion/decision-record.md:39-40`, `:55-58`

**Issue:**
D1 and its Bound exclude `--by arbiter` and `--by parent` entries and count `--by operator`.
`internal/loomcli/decision.go` accepts `parent`, `operator` and `arbiter` and refuses only a task session's label and a non-arbiter `--by arbiter`; its own comment says `--by` stays an unverified label.
The fork is a fork of the parent session, so it can pass `--by operator`, and so can the orch.
Round 2 closed the arbiter and parent routes into a settle, and this route stays open with no stated bound, which the rubric's attack-surface question asks for.

### [MEDIUM] Board entry and board note count without a provenance test

**Location:** `_lyx/discussion/decision-record.md:39`

**Issue:**
The definition lists "the task's board entry" and "an operator's board note" as recorded operator decisions.
This task's own entry holds `Operator decision (2026-10-16)` sections beside `Agent's suggestions` and an `Operator view, not a decision`, so the whole entry is not a decision source.
A board note has no author field, and agents file notes too, so "an operator's" has no test the fork can apply.
An agent suggestion would count as an operator decision, which is the failure round 2 closed for the arbiter, parent and auto-pick routes.

### [LOW] The support log is a decision source that the brief never hands the fork

**Location:** `_lyx/discussion/decision-record.md:39`, `:48`

**Issue:**
The definition counts a Discussion answer that the support log's `## Interview` shows, while the brief's materials list names the decision record at `{{.decision_record_path}}` and no support log.
A fork following the brief reads the record only, so the source is unreachable by the brief's own list.

### [MEDIUM] A role bullet that has the orch decide a design call survives beside D1 and D2

**Location:** `_lyx/discussion/decision-record.md:60`

**Issue:**
`contracts/stencils/orch/orch-template-role.md:67`, "A design call the child cannot make: decide it and record it with `lyx loom decision add`", sits in `## Escalations from a child`, which D1 rewrites.
D1 keeps "the other mechanics bullets" without naming it, and D1 and D2 forbid the orch to decide a review escalation as a design call and send an unanswered question to the operator.
A rendered role file would carry both rules for a review escalation; the Acceptance criterion on the rendered role file does not catch it.

### [LOW] Decide fragments still frame the choice as the fork's own decision

**Location:** `_lyx/discussion/decision-record.md:45-55`

**Issue:**
`bouncer-fragment-decide-open.md` and `bouncer-fragment-decide-capped.md` both begin "Record your decision on the segment", and D1's Bound lets the fork run exactly those options.
D1's step 1 change tells the fork to record an applied operator decision, not its own.
The fragments are separate stencils outside Scope's file list, so the contradiction stays unplanned.

### [LOW] D9's perch owner does not hold at a merge budget of 0

**Location:** `_lyx/discussion/decision-record.md:182-183`, `:237`

**Issue:**
Checked against the code: the Merge-Review perch runs on a clean merge-in, because `landingshed.MergeRecord.SkipsReview` skips only an up-to-date merge or one with no shared file, package or pattern change, and a parent edit to `CLAUDE.md` against the task's own edit shares a file.
The perch's `Merge-Burler-Fix` has `fix-scope: source`, which allows editing `ORCHESTRATOR.md`.
But `merge_max_bounces: 0`, a `loom.yaml` or `run_settings` key, makes the Bouncer settle with no review, and D9 and its Acceptance criterion claim the check without that condition.

### [LOW] D9's lands-first note has no named reader

**Location:** `_lyx/discussion/decision-record.md:184-186`

**Issue:**
The board shows `reed-add-parentless-placement` at `Plan-Burler-Review`, ahead of this run, and its scope removes only the `CLAUDE.md` workaround.
Its perch measures merges against its own decision record, not the board, so a note on the board reaches no agent of that run once its plan is written.
D9 rationale says the note is a Decision so the plan writer schedules it, which it does, but no actor is named to act on the note when that run lands, and the `ORCHESTRATOR.md` entry survives.
