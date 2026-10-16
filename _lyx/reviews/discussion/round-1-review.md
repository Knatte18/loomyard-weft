---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D4, close-as-noise and fold outcomes)
    summary: D4 lets a duplicate of an existing entry be either folded or closed as noise, with no rule choosing between them, and the close drops what a fold would carry
    fix: 'In D4, replace the close-as-noise bullet with "A close as noise: only an issue that adds nothing to the recorded issue or entry it duplicates, or an issue with no actionable content, with a reason naming which; a duplicate that adds anything to the entry is folded into it, or becomes a new note when the entry is run-held."'
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D1 and Acceptance criteria, third bullet)
    summary: D1 and the acceptance criteria give the empty inbox as a bare [] while the output shape is an issues object, so a test could pin a top-level array
    fix: In D1, change "an empty inbox as `[]`" to "an empty inbox as `{"issues":[]}`"; in Acceptance criteria, change "Both modes print `[]` for an empty inbox." to "Both modes print `{"issues":[]}` for an empty inbox."
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D4, close re-run paragraph; D3)
    summary: The close re-run has no count and no outcome when it fails again, and a recorded issue left open is hidden from list, so no later wake recovers it and nothing says what the orch does with a failed line
    fix: In D4, change the re-run sentence to say the fork re-runs the `close` payload once, accepting a second pointer comment when the first one posted, and that a second failure gives the issue a `failed` line carrying the error and the `close` payload; in D3, add a bullet that a `failed` line from the intake fork is handled as a lyx bug under Where a finding goes, naming the issue and the payload.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D4, Bound paragraph)
    summary: The bound grants the label write of upsert without confining it to the entry just folded into or to adding labels, though upsert's labels replace the whole list
    fix: In D4's Bound paragraph, change "and the label write of `lyx board upsert`" to "and a `lyx board upsert` that writes only the labels of an entry it just folded an issue into, keeping every label the entry carries and adding the issue's module labels".
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md (Scope In, fifth bullet; Acceptance criteria)
    summary: The In item for internal/boardcli/doc.go has no Decision or acceptance criterion
    fix: In Acceptance criteria, extend the last bullet with "and `internal/boardcli/doc.go` states D1's list output, the `--full` flag and D2's import output".
---

### [MEDIUM] Close-as-noise and fold overlap for a duplicate of an entry

**Location:** `_lyx/discussion/decision-record.md`, D4.

**Issue:** D4 allows a fold "only when the issue is essentially the same finding as that entry", and a close as noise for "a duplicate of a recorded issue or entry".
An issue that duplicates an entry meets both conditions, and D4 gives no rule choosing between them.
The orch's board policy (`contracts/stencils/orch/orch-template-role.md`, The board) says a finding essentially the same as an existing note goes into that note, so the close branch lets the fork skip that policy.
The two outcomes differ in what survives: a fold carries the issue body and comments into the entry, while a close carries nothing to the board, and a closed issue can no longer be imported (`boardengine` refuses a closed issue until it is reopened on GitHub).
D4 also says a fold into a run-held entry becomes a new note, but the close branch offers a second exit for the same issue that loses its content.
This is the attack-surface bound the focus asks about: the close is the one write the fork cannot undo, so its bound has to rule out the lossy choice when the issue adds anything.

### [LOW] Empty-inbox shape disagrees with the stated output shape

**Location:** `_lyx/discussion/decision-record.md`, D1 and Acceptance criteria.

**Issue:** D1 gives the output as `{"issues":[{"number":…,"title":…}]}` and then "an empty inbox as `[]`", which reads as the empty value of `issues`.
The acceptance criterion "Both modes print `[]` for an empty inbox" states the whole output as `[]`.
Today `intakeList` prints `{"issues":[],"ok":true}` for an empty inbox (`internal/boardcli/intake.go`, `output.Ok`), and the orch reads that shape, so a top-level array is not intended, but the criterion as written would let a plan or test pin one.
The non-empty shapes of D1 and `--full` agree with each other and with today's output.

### [LOW] The close re-run has no bound and no failure route

**Location:** `_lyx/discussion/decision-record.md`, D4 and D3.

**Issue:** D2's and D4's envelope fields agree with the acceptance criteria: four short fields plus `close`, and no `entry`.
D4 then has the fork re-run the `close` payload, with no count and no outcome when that fails too.
`selfreportengine.CommentAndClose` posts the comment before closing, and its error says that re-running posts a second comment, so each re-run after a posted comment adds a pointer comment.
A re-run cannot double-close: `intake close` refuses an already closed issue.
When the re-run also fails, the issue stays open but recorded, and `intake list` filters recorded issues, so D3's rule that a later wake forks again never reaches it.
The return line has a `failed` outcome, but neither D3 nor D4 says what the orch does with one, so that orphan is reported once and then dropped.

### [LOW] The upsert label write is not confined in the Bound paragraph

**Location:** `_lyx/discussion/decision-record.md`, D4 Bound.

**Issue:** The Bound paragraph lists "the label write of `lyx board upsert`" as one of the fork's two write routes, with no limit on which entry it targets or which labels it may remove.
The fold bullet implies the narrower use (add the issue's module labels to the fold target), and D5 asks the skill to say the upsert carries the whole label list, but the bound itself, which is what the stencil states as the fork's limit, lets a label rewrite of any unheld entry through, type label included.
New notes need no upsert, since D5 routes their labels through `import`'s `labels`.

### [LOW] The doc.go In item has no Decision or acceptance criterion

**Location:** `_lyx/discussion/decision-record.md`, Scope and Acceptance criteria.

**Issue:** Every other In item maps to a Decision: the list to D1, import to D2, the stencil to D3 and D4, the skill to D5.
The fifth In item, `internal/boardcli/doc.go` stating the new list and import output, appears in no Decision and no acceptance criterion.
Today's `doc.go` names the intake verbs but states neither output, so this is new text the plan writer has no criterion to meet.

## Verdicts on the focus directives

- D1 against Acceptance criteria: the non-empty shapes agree with each other and with today's `{"issues":[...]}`; the empty case is F2.
- D3 fork-open rule: not a defect.
  The orch spawned the fork in its own context and knows it is open until its report arrives, as Parent-review's "a repeat notice for a request you already forked for starts no second fork" already relies on.
  The orch-note bullet covers the case after a context cycle, when that knowledge is gone, and the CLI settles each issue once, so the two bullets cover disjoint cases.
- D4 attack surface: `intake import` as a new note is bounded by the type and module label rule, and a fold by the board policy and the run lock; the close is F1 and the upsert label write is F4.
- D2 and D4 envelope: the fields agree; the re-run's bound is F3.
- D5 and Constraints: the label-list-carrying upsert rule is in D5 and the fold-label rule in D4, so the decision record holds both.
  The support log's other content is deliberation: the noise-close race between two forks is recorded as the fork's own judgment, not a requirement, and the run-lock premise was rejected with a reason.
- Scope against Decisions: doc.go is F5.
  No Decision edits this task's own board entry; D3 to D5 change a stencil, a skill and `boardcli`.
