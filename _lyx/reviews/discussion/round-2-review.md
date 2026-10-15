---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D2)
    summary: D2 puts an agent-workflow rule with no code anchor into PATTERN.md, whose package doc limits PATTERN text to rules for the code
    fix: In decision-record.md, drop D2 and the acceptance criterion `PATTERN.md carries the new entry.`, and drop the PATTERN headroom-lint Constraint and the lint-pattern criterion with it; extend D1 so the `CLAUDE.md` Docs-land rule itself states that no decision, card or step of a run changes the running task's own board entry's body or scope, and move D2's rework/prior-plan rationale onto D1 (every Plan writer, darn writer and rework session loads `CLAUDE.md`, as D4 and D5 already rely on). In support-log.md, record under Rejected alternatives that a PATTERN entry was rejected because `internal/pattern/doc.go` says PATTERN text states rules for the code only, and amend the Turn 4 pattern-fit line that added D2.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D5, D6)
    summary: D5 lets the darn writer drop a decided step before the record carries the drop, with no no-parent branch, unlike D4, and D6's darn-review line then flags that interim drop
    fix: Rewrite D5's second sentence to mirror D4 - the darn writer asks its parent to record the drop with `lyx loom decision add`, leaves the step out only once the record carries that `Added after Discussion` entry, and until then follows the darn stencil's existing unrecorded-correction rule (names it as an unrecorded parent correction in the change description's design decisions); with no parent recorded it stops and reports. Add that the route admits a drop of an own-entry step only, as D4 does, so the attack-surface bound is stated in D5 itself.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D1, D3, D4, D5, D6)
    summary: The Goal narrows editing the own entry to a body or scope change, but the texts D1 and D3-D6 order written say only edit the own board entry, which outside the record also forbids status and run_settings writes the run lock allows
    fix: In D1, D3, D4, D5 and D6, state that the rule text written into `CLAUDE.md`, `docs/overview.md`, each stencil and each rubric says changes the task's own board entry's body or scope, as D2's entry already does, so the written text forbids no more than `internal/boardengine/runlock.go` refuses and the landing's done status stays outside it.
---

### [MEDIUM] PATTERN entry for a workflow rule

**Location:** `_lyx/discussion/decision-record.md`, D2.
**Issue:** `internal/pattern/doc.go:81-82` says "PATTERN text states rules for the code only: it never says how an agent, a run, a stencil or a workflow uses it", and `PATTERN.md` opens as "The structural invariants of the loomyard code".
D2's entry (`Planning or doing a task's work: no decision, card or step changes the running task's own board entry's body or scope`) governs decisions, cards and steps, all agent artifacts, and names no code the rule binds; the board lock that does bind code is out of scope and unchanged.
D2's rationale confirms the purpose is reaching agents through the inlined overview, which uses PATTERN as an instruction channel.
Existing entries that touch workflow (`PATTERN-review-round`, `PATTERN-role-skills-typed`) each anchor in Go behavior; this one has none.
The rule still needs a home every role reads, and `CLAUDE.md`, which D4 and D5 already say the Plan and darn writers load, carries it without the overview budget cost.
Completeness check: the rule's requirement moves into D1, so nothing the plan writer needs is lost.

### [MEDIUM] Darn drop is unbounded before recording

**Location:** `_lyx/discussion/decision-record.md`, D5 and D6.
**Issue:** D4 lets the Plan writer plan without the step only once the record carries the `Added after Discussion` entry, and stops with no parent.
D5 has the darn writer leave the step out at once and ask its parent afterward, with no branch for a run with no parent and no rule for the interval before the record carries the drop.
The darn stencil's existing route (`contracts/stencils/darn/darn-template-write.md:78-81`) is parent-initiated, and for a correction the record does not yet hold it requires naming it as an unrecorded parent correction in the change description; D5 calls it the "existing scope-correction route" yet uses it in the opposite direction and skips that requirement.
D6's darn-review line accepts the drop only "once the record carries that entry", so a drop the parent never records leaves the darn diff failing review with the only fix being the parent's (`loom-rubric-darn-review.md:54`).
Under the rubric's attack-surface question, D5 introduces a routing edge whose bound is stated only for the Plan writer.

### [LOW] Rule wording wider than the lock

**Location:** `_lyx/discussion/decision-record.md`, D1 and D3 to D6.
**Issue:** `internal/boardengine/runlock.go` refuses a held entry's removal or any field change but `Status`, `Created`, `Revised` and `RunSettings`.
The Goal matches that ("body or scope") and defines "editing the own entry" as such a change for the record.
The texts the decisions order written do not carry that definition: D1 has `CLAUDE.md` say the task "never edits its own board entry" in the same sentence as "the landing marks the entry done", and D3, D4, D5 and D6 say "edit this task's own board entry".
Read in a stencil or in `CLAUDE.md`, without the record's Goal, each forbids a status or `run_settings` write the lock allows.
D2's entry text is the one that says "body or scope".

## Focus verdicts

- Overview budget: not a defect.
  `HeadroomBytes` is 16384 × 95 / 100 = 15564, and `PATTERN.md` is 15560 bytes on this branch with no PATTERN change against `main`.
  `patternlint` fails only when the head is longer than both the base and the line; D2's cut of at least the entry's own bytes keeps the head at or under the base, and the D2 entry line measures 212 characters, under `MaxEntryLineChars` (320).
  The Constraint and the `lint-pattern --base` criterion pin it; F1 removes the need.
- Own-entry drop route: D4 states who records (the parent, via `lyx loom decision add`), when the Plan writer may drop (once the record carries the entry) and the no-parent case (stop and report). Not a defect.
- Plan-review acceptance: D6 names what it lets through (a missing own-entry step only) and the on-disk evidence (an `Added after Discussion` entry recording the drop). Not a defect.
- Darn path: F2.
- Rule wording against the lock: F3.
