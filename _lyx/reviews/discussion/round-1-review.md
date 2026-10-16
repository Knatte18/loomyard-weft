---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5.1, D5.2, D5.4
    summary: A declared PATTERN change downgrades the never-downgrade PATTERN guard for the declared entry, and no Decision states that bound or checks that the declared text ever lands in PATTERN.md
    fix: In D5.4, add a Bound line stating what a declaration lets through (every reviewer, judge and implementer follows and judges the listed entry by the declared text, never PATTERN.md's current text) and what bounds it - only the entries listed in the record's `## PATTERN changes`, only entries a bullet of the D5.1 shape names, appended after Discussion only through `lyx loom decision add`'s existing `--by parent|operator|arbiter` caller guards (state this in D5.2's Bound too). Add one Decision (D5.7) that closes the loop on landing - the Webster-Review rubric checks that the branch's `PATTERN.md` and `pattern/` diff carries each declared entry's add, change or remove with the declared text, and a declared entry the diff does not land is a BLOCKING finding - and add the matching Acceptance criterion.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2.8
    summary: D2.8 makes the verify-fix fork's raw reply the Master's source for fix commits and departures, against the board entry's operator decision that the Master never reads raw fork output and a departure reaches it through Go
    fix: Replace D2.8 with a Go route - the verify gate closure, which already walks every commit from the recorded pre-fix head to HEAD on the first-parent chain (`fixCommitRejection`), also collects each `fix:` commit's SHA and subject and each `Pattern-Departure:` trailer, and hands them to the Master in the gate result it re-arrives with; the summary's `## Verify gate fixes` section takes the commits and departures from that result. Drop the master stencil's named exception, update D2.4's field list and the Acceptance criteria to match, and move the reply-as-exception option to support-log.md Rejected alternatives with the operator decision as the reason.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2.6
    summary: D2.6's bound rests on a false premise - the plan-level verify gate runs the plan's `## verify:` command, not the whole suite, and a plan with no `## verify:` section passes it with a warning, so a misjudged uncaused failure can pass the run uncaught
    fix: In D2.6, rewrite the Bound to name the plan's `## verify:` command instead of the whole suite, and make the OK-on-uncaused-failure rule apply only when the plan declares a `## verify:` section; when it declares none, a batch gate failure stays `FAILED` as today. State how the fork learns which case holds (a value rendered into `webster-body-implementer.md` from the plan), and add an Acceptance criterion for the no-verify-section case.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D5.4
    summary: D5.4's spawn refusal on a malformed record names `lyx loom validate-discussion` as its way forward, which only diagnoses and makes no progress from the refused state, and burlerengine, one of the refusing callers, has no refusal-spec.md section
    fix: In D5.4, make the way forward one refusal-spec.md accepts - fix the named section of `_lyx/discussion/decision-record.md` (the run's parent or the operator, since no later role's write surface holds the record), confirm with `lyx loom validate-discussion`, then re-step the row - and state which refusal-spec.md section each caller's row goes in, placing burler's under the loom row whose segment spawns it, or say the burler caller propagates the error to that loom row.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1.4
    summary: D1.4's sentence that no other role and no other card runs a tool that rewrites files contradicts the edit directive's existing `lyx loom fmt <file>` allowance and D1.6's `git mv` for a file pair
    fix: In D1.4, word the exception's second half as - no other role and no other card runs a script or tool that rewrites file content beyond what this directive already allows - so `lyx loom fmt` and `git mv` stay covered.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Scope Out, fourth bullet
    summary: The Scope Out bullet on landing a PATTERN edit during Discussion carries deliberation narrative, and its one requirement, the operator's ruling that the PATTERN edit itself lands in Webster, is stated in no Decision
    fix: Add to D5.1 the sentence - the PATTERN edit itself lands in a Webster card, never during Discussion or Plan (operator decision). Cut the Scope Out bullet to - Landing a PATTERN edit during Discussion (`discussion-lands-invariant`). Move the clause from D5 answers the same need onward to support-log.md Rejected alternatives.
---

### [BLOCKING] A declared PATTERN change is an unbounded guard downgrade

**Location:** `_lyx/discussion/decision-record.md` D5.1, D5.2, D5.4

**Issue:** The reviewer and judge directives make every cited PATTERN violation BLOCKING and never downgradable.
D5.4 renders a rule that an entry listed in the record's `## PATTERN changes` is followed and judged by its listed new text, never the old, so a declaration switches that guard off for the declared entry's current text.
That is a guard made bypassable, and the rubric's attack-surface question asks the Decision to state what it lets through and what bounds it.
D5.2's Bound covers only the verb's file mechanics (appends one bullet, removes nothing).
It says nothing about who may declare: `lyx loom decision add` is run by the parent, the operator and the arbiter (refusal-spec rows 316 and 319), so an arbiter session can disarm a PATTERN entry for every later reviewer.
Nothing checks that the declared text lands in `PATTERN.md`: D5.3 matches the plan to the record, but no Decision or Acceptance criterion matches the branch's PATTERN diff to the declaration.
A task that declares a change and never makes it lands code that violates the PATTERN in force after landing, with every review having judged it against text that never existed.

### [BLOCKING] D2.8 contradicts the operator's departure route

**Location:** `_lyx/discussion/decision-record.md` D2.8

**Issue:** The board entry's second operator decision under F12/F13 reads: the Master still never reads raw fork output; the departure reaches it through Go.
D2.8 instead names the verify-fix fork's final reply as an exception the Master reads, and routes that fork's `Pattern-Departure:` trailers through it.
The rationale, the Master runs no git, rules out the Master reading commits, not Go reading them.
`internal/websterengine/doc.go`'s verify gate section shows the closure already walks every commit from the recorded pre-fix head to HEAD on the first-parent chain (`fixCommitRejection`), so the Go route costs one more pass over commits it already reads.
The support log's rejection of a Go-side list (more plumbing) weighs cost against an operator ruling and cannot override it.

### [MEDIUM] D2.6's bound names a catch that may not exist

**Location:** `_lyx/discussion/decision-record.md` D2.6

**Issue:** The Bound says a misjudged failure is caught by the plan-level verify gate, which runs the whole suite.
`internal/websterengine/doc.go` says the gate runs `verifytree.Verify` over the plan's `## verify:` command, and that a plan with no `## verify:` section passes with a warning.
So the bound holds only for a plan whose verify command covers the failing test; otherwise D2.6 lets a batch-gate failure the fork misjudged as uncaused through the whole run.
The Decision's stated bound is therefore wrong, which the rubric's attack-surface question treats as no bound.

### [MEDIUM] D5.4's refusal names a diagnostic, not a way forward

**Location:** `_lyx/discussion/decision-record.md` D5.4

**Issue:** `contracts/specs/refusal-spec.md` defines a way forward as a `lyx` verb that makes progress from the refused state, a transition the run takes by itself, or a plain git command.
`lyx loom validate-discussion` only reports findings; re-running the spawn after it still refuses.
After Discussion, no spawned role's write surface holds the decision record, and `decision add` restores the record when its own append fails the check (refusal-spec row 320), so the message has to name who fixes the record and how the run resumes.
D5.4 also lists `burlerengine` among the callers, and burler has no section in `refusal-spec.md`, so "with its `refusal-spec.md` row" has no home for that caller.

### [LOW] D1.4's exception wording outlaws allowed rewrites

**Location:** `_lyx/discussion/decision-record.md` D1.4

**Issue:** `contracts/stencils/edit/edit-directive.md` already allows `lyx loom fmt <file>`, which rewrites a Go file, and D1.6 has a file pair use `git mv`.
"No other role and no other card runs a tool that rewrites files" read literally forbids both, so the directive would contradict itself once the exception lands.

### [LOW] Scope Out carries a requirement and deliberation

**Location:** `_lyx/discussion/decision-record.md` Scope Out, fourth bullet

**Issue:** Applying the completeness-before-leanness test first: the bullet's clause "the edit stays in Webster" is the operator's ruling (the edit itself still happens in Webster, not before), and no Decision states it, so the plan writer has it only inside a Scope Out justification.
The rest of the clause, why D5 answers the need `discussion-lands-invariant` raises, is deliberation narrative that belongs in the support log.

## Settled focus questions with no finding

- D1.4 and D5.5: `PATTERN-edit-directive`'s entry and background state that the `edit-directive` stencil is the rule's only statement and that every spawned role's opening stencil renders it; they do not state the rule's content.
  Adding the exception to the stencil keeps the stencil the only statement, so it edits no PATTERN entry and D5.5 holds.
  D1.4 states its bound and that the bound is prose only; F5 is a wording defect, not an unbounded exception.
- D2.5: the Bound states what a trailer excuses (only the difference it names) and what bounds it (the cited entry, in its declared new text, must forbid the card's way).
  Not a defect.
- D1.8: a new refusal, not a weakened guard; it names a way forward (set `rename_tools` from the prime, or recast the card), the master rung writes `outcome: stuck`, and `recover-batch` refuses the same way, so a run is not stranded.
  Not a defect.
- D2.3, D2.8 and D5.3 against Constraints: D2.3's read goes through `internal/gitrepo` behind webster's `Git` seam as the Constraints state, and D5.3 reads the record's section through `discussionparser` as `PATTERN-sole-parsers` requires.
  D2.8's conflict is with the board's operator decision, filed as F2, not with the Constraints.
- No Decision edits this task's own board entry's body or scope.
