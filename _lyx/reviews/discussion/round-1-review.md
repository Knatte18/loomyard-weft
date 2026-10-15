---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D2, Constraints)
    summary: D2's new PATTERN.md entry pushes the overview past its headroom line, so every card gate and the Webster-Burler-Fix gate refuse the branch, and the record names no cut to pay for it
    fix: In D2, add that the new entry lands with a cut of at least its own byte count elsewhere in PATTERN.md, by moving detail from existing entry lines into their background files, so the overview ends at or under the headroom line (95% of `pattern.MaxOverviewBytes`); add a Constraints bullet naming the PATTERN headroom lint (`lyx loom lint-pattern`, run by every card gate) and that the branch's overview must not cross the line; add an Acceptance criteria bullet that `lyx loom lint-pattern --base <merge base>` passes.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D4)
    summary: D4 routes an own-entry step through the Plan stencil's existing route, but that route covers only a scope addition arriving after Discussion and says nothing about dropping a step the record already holds, so D4 rests on a route that does not carry a drop
    fix: Rewrite D4's second sentence to say the Plan stencil's post-Discussion route (`contracts/stencils/loom/loom-template-plan.md`, the paragraph opening "A scope addition that arrives after the Discussion") is widened to name a drop of a decided own-entry step, that the parent records the drop with `lyx loom decision add` as an `Added after Discussion` entry, that the Plan writer plans without the step only once the record carries that entry, and that with no parent recorded it stops and reports; state in D4 that the widened route admits a drop of an own-entry edit only and no other decided step.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6's Plan-review line accepting a dropped step downgrades the rubric's every-Decision-carried guard without stating its bound, so a plan that drops an own-entry step with no recorded drop reads as accepted
    fix: In D6, replace "The Plan-review line accepts a step dropped through D4's route." with a sentence stating that the Plan-review line accepts a missing step only when the step edits this task's own board entry and the decision record carries an `Added after Discussion` entry recording its drop, and that any other uncarried Decision stays a finding as before.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Goal, D2)
    summary: The rule forbids any edit of the task's own entry, while the board lock refuses only scope edits and the run's own Finalize row marks the entry done, so the Goal and the PATTERN text as worded also cover the landing's status write
    fix: In the Goal and in D2's proposed PATTERN text, replace "edit(s) ... own board entry" with "change(s) ... own board entry's body or scope", and keep D2's closing clause as "its landing marks it done and the board prune removes it", so the rule matches what the lock refuses and leaves the landing's status write outside it.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D5)
    summary: D5 gives the darn writer the rule but no route for a board entry or record that asks for an own-entry edit, while the darn writer and darn-review take the board entry as the measuring stick, so a dropped step has no recorded basis
    fix: Extend D5 to say that when the board entry or the record asks for a step that changes this task's own board entry, the darn writer leaves the step out and asks its parent to record the drop through darn-template-write's existing correction route (`lyx loom decision add`), and that D6's darn-review line accepts a step dropped once the record carries that entry.
---

### [BLOCKING] D2's PATTERN entry crosses the overview headroom line

**Location:** `_lyx/discussion/decision-record.md`, D2 and Constraints.
**Issue:** `PATTERN.md` is 15560 bytes at HEAD.
`internal/pattern/check.go` sets `HeadroomBytes = MaxOverviewBytes * 95 / 100`, which is 15564 for the 16 KiB cap, so the overview has 4 bytes of headroom left.
D2's proposed entry line is well over a hundred bytes, so the branch grows the overview from under the line to above it, the case `TestHeadroom` pins as a finding.
`internal/loomshed/verifygate.go` runs `patternlint.Lint` in every card gate, and `refusal-spec.md` names the Webster-Burler-Fix verify gate refusing on it, so a plan following D2 as written cannot pass its gates.
The record carries no constraint or decision that pays for the bytes.

### [MEDIUM] D4 leans on a route that does not carry a drop

**Location:** `_lyx/discussion/decision-record.md`, D4.
**Issue:** The stencil's route in `contracts/stencils/loom/loom-template-plan.md` reads "A scope addition that arrives after the Discussion is a design call the parent records", followed by planning on it once the record carries it.
A decision asking for an own-entry edit is not an addition and does not arrive after Discussion: the record already holds it.
D4 calls this "the stencil's existing route for a scope change" and states the Plan stencil only gains the no-card rule, so nothing in the decided text tells the Plan writer the route applies to a drop, and D4 does not name how the parent records one.
The routing edge's bound ("bounded to own-entry edits") sits only in the Rationale, not in the text the stencil is to carry.

### [MEDIUM] D6's Plan-review acceptance has no stated bound

**Location:** `_lyx/discussion/decision-record.md`, D6.
**Issue:** `contracts/stencils/loom/loom-rubric-plan-review.md` requires every Decision and every Constraint in the decision record to be carried by some card.
D6's Plan-review line weakens that guard for a dropped step and states only "through D4's route" as its bound, which inherits F2's gap.
The attack-surface rule asks the Decision to state what the downgraded guard now lets through and what bounds it: here, which steps may go missing and what evidence on disk the reviewer checks.

### [LOW] "Edit" is wider than what the lock refuses

**Location:** `_lyx/discussion/decision-record.md`, Goal and D2.
**Issue:** `internal/boardengine/runlock.go` refuses a held entry's scope edits, and `internal/boardcli/doc.go` lists the scope-changing verbs.
The run's own Finalize row marks the board task `done` (`docs/overview.md`, landing section), which is a write to the task's own entry by a step of its run.
The Goal ("No step of a task's run is asked to edit that task's own board entry") and D2's "no decision, card or step edits" read literally as covering that write.

### [LOW] D5 gives darn no route for a requested own-entry step

**Location:** `_lyx/discussion/decision-record.md`, D5.
**Issue:** `contracts/stencils/darn/darn-template-write.md` has the darn writer read the board entry as its spec, and `loom-rubric-darn-review.md` measures the diff against the board entry as amended by the record's `Added after Discussion` entries.
D4 gives Plan a recorded route for an own-entry step it must not carry out; D5 gives darn only the prohibition, so a darn writer facing such a request drops it with no record, and the darn reviewer measuring against the entry sees an unexplained gap.

## Focus verdicts

- D4 and the Plan stencil's parent route: the route is worded for an addition and does not carry a drop; the bound is stated only in the Rationale (F2, F3).
- Other statements of the documentation-lifecycle rule: `CLAUDE.md:47` and `docs/overview.md` lines 95-96 are the only texts that move a design out of the board entry, and D1 covers both.
  `PATTERN.md`'s `PATTERN-documentation-lifecycle` line, its background file and `docs/overview.md`'s "Planned-but-not-built work lives on the board" line say only where a design lives; `plugins/ly/skills/board/SKILL.md` says nothing about moving a design.
  No source of the defect is left; not a defect.
- D2's PATTERN text: it is scoped to "the running task's own board entry" and forbids no write to another entry or board-module change; its verb is wider than the lock (F4), and the entry does not fit the overview's headroom (F1).
- D6 and Scope Out: the Discussion-review and darn-review lines add guards and remove none; the Plan-review line is the one downgrade and lacks a stated bound (F3).
  The darn route gap is F5.
  The Scope Out rejection of a `validate-plan` check removes no existing guard; not a defect.
