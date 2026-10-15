---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:49-52
    summary: D4 says the darn writer leaves the own-entry step out only once the record carries the entry, then says that until then it names the drop as an unrecorded parent correction, which presupposes the step is already left out
    fix: In D4, replace the sentences from "When the board entry or the record asks" through "With no parent recorded it stops and reports." with a single ordered account. The darn writer leaves the own-entry step out from the start, because the run lock refuses the edit anyway. It asks its parent to record the drop with `lyx loom decision add`. Until the record carries the `Added after Discussion` entry, it names the omission in the change description's design decisions as an unrecorded parent correction, which the darn-review rubric files as a MEDIUM `decision` finding deferred to the parent. Once the entry exists, the omission is no deviation from the amended board entry. State that it stops and reports only when it has no parent to ask. Keep the closing sentence that the route admits a drop of an own-entry step only. Then reword D5's darn-review sentence to match, and add a one-line note that the darn route leaves the step out at once where the Plan route waits, because a darn run is one autonomous session that cannot wait on a reply.
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:5-6
    summary: The rule text is ordered as the entry's body or scope, but the record never says what scope covers or that run_settings is outside it, so a writer can read run_settings as scope
    fix: In the Goal, after the sentence that the landing's status write is outside the own-entry edit, add that the entry's `run_settings` are outside it too, and that scope means the fields the run lock compares, such as title, kind, recipe, priority, isolated, brief, short name, labels, issues and dependencies. In Acceptance criteria, add `run_settings` to the allowed writes in the criterion that opens "Every new rule is scoped to the body or scope". Do not copy the field list into the stencil text; the stencils keep the phrase "body or scope".
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:27-31
    summary: D1 orders the CLAUDE.md rule without ordering its rationale into CLAUDE.md, so the one home every role reads states the rule with no reason
    fix: In D1, add to the sentence that orders the CLAUDE.md rule a short because-clause naming the reason the text carries, that the run's board lock refuses a change to the running task's own entry. Keep the existing rationale paragraph for the plan writer; it explains why the home is CLAUDE.md and says nothing about what the CLAUDE.md text itself states.
---

### [MEDIUM] D4 contradicts itself on what the darn writer does before the record carries the drop

**Location:** `_lyx/discussion/decision-record.md:49-52`

**Issue:** D4 reads "leaves the step out only once the record carries that `Added after Discussion` entry".
The next sentence reads "Until then it ... names the drop as an unrecorded parent correction in the change description's design decisions."
Naming a drop in the change description presupposes the step was left out, so the two sentences describe opposite behavior for the same interval.
The darn stencil's existing rule at `contracts/stencils/darn/darn-template-write.md:80` is "follow the message meanwhile", which means act on the correction before it is recorded.
The darn-review rubric's unrecorded-parent-correction line files that case as MEDIUM `decision` and defers it to the parent.
D5 matches the second reading ("before then, the drop is an unrecorded parent correction under the rubric's existing rule").
D4's first reading would have the writer make the edit meanwhile, and the run lock then refuses the write.
The plan writer implementing D4 cannot tell which sentence wins.
The existing rule is also written for a parent message that carries the correction, and here no message exists: the board entry or the record asks for the step, and the writer asks.
D4 should say that the writer applies the rule's consequence on its own initiative.
The Plan route (D3) differs: the Plan writer plans without the step only once the record carries the entry.
The difference is defensible, because a plan writer can stop and wait while a darn run is a single autonomous session, but the record states neither the difference nor its reason.

### [LOW] The record does not pin what scope covers or where run_settings sits

**Location:** `_lyx/discussion/decision-record.md:5-6`

**Issue:** `internal/boardengine/runlock.go` `scopeChanged` compares title, kind, recipe, priority, isolated, brief, body, short name, labels, issues and dependencies.
It never compares status or `RunSettings`, and its comment says a run settings change is not a scope change, so a run-held entry takes it.
The Goal and the last acceptance criteria name only "the landing's status write" as outside the rule.
The rule text the stencils carry says "body or scope", and plain reading of "scope" can take in run settings such as the driver or isolation.
A stencil that forbids that would forbid more than the lock refuses, which the record's own constraint rules out.
The record should say that run settings are outside the rule, and that scope means what the lock compares.
It should not copy the field list into the stencils.

### [LOW] D1 orders the CLAUDE.md rule without its rationale

**Location:** `_lyx/discussion/decision-record.md:27-31`

**Issue:** D1 orders the sentence "no decision, card or step of a run changes the running task's own board entry's body or scope" into `CLAUDE.md`.
The rationale paragraph explains why `CLAUDE.md` is the home and why the old sentence misled, but nothing orders the reason for the rule into the text.
Placement itself holds: `CLAUDE.md` is auto-loaded by every session in the worktree, `internal/pattern/doc.go` limits PATTERN text to rules for the code, and the stencils each state the rule for their own role as `PATTERN-producer-pointer-rule` asks.
The rule stands once in `CLAUDE.md` and once per role stencil, which is the intended split.
A reader of `CLAUDE.md` alone gets the prohibition without the lock that backs it, and the lock is also what bounds the rule to body or scope.

## Rejected

- The Plan stencil does not name `CLAUDE.md`, only the darn stencil does, so D1's claim that the Plan writer loads it might look unsupported. Rejected: the harness auto-loads the worktree's `CLAUDE.md` for every spawned session, so the claim holds without a stencil instruction.
- The `CLAUDE.md` text and the stencil texts say the same rule in several homes. Rejected: each home is a different reader (all roles, Discussion writer, Plan writer, darn writer, three reviewers), so this is not a duplicate within one file.
- D3's Plan-review line accepts a missing step only on a recorded drop. Rejected as a finding: it is bounded to own-entry steps and any other uncarried Decision stays a finding.
