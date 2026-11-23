---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shedadapters/bouncerdecision.go#routeDecisions
    summary: "At a spent budget, a Code-Review round whose review holds a decision finding and an open BLOCKING finding of another class escalates with cause decision, which the arbiter may settle; an arbiter accept then passes the open BLOCKING findings through acceptRound without the parent, against goal 2."
    fix: "In routeDecisions (internal/shedadapters/bouncerdecision.go), before escalating an undecided round with cause decision, check BlockingEscalates the way settleUndecided does: when the soft-cap window is open (budgetReached or atHardCap) and the round's ledger holds an open BLOCKING entry whose class is not decision, or cannot be read, hand the round to the parent with cause budget instead (escalateWith(round, EscalationBudget, detail, ptr), the detail naming both the open BLOCKING keys and the decision findings). A budget escalation is parent-only under arbiterstop.Eligibility, and the parent's accept or continue is then acted on by settleUnconverged. Add a TestBouncer_AutoAccept or decision-escalation row with escalateDecisions and blockingEscalates set, a spent budget, a review holding one decision finding and a ledger with an open non-decision BLOCKING key, expecting Awaiting with cause budget."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/loomengine/template.yaml:53
    summary: "The diff changes what code_auto_accept true means (it no longer accepts an open BLOCKING finding at a spent budget) but ships no configreg.Reinterpretation, which PATTERN-config-layers requires for a meaning change; the change description also calls the door two-way and claims the key keeps its meaning."
    fix: "Add a configreg.Reinterpretation on the loom module (internal/configreg/configreg.go, with its Rewrite in internal/loomengine) whose Rewrite flags a hub base file that sets code_auto_accept: true, naming the way forward: such a round now escalates to the run's parent with cause budget when a BLOCKING finding is open, and the value stays as it is. In _lym/landing/summary.md, drop the claim that code_auto_accept keeps its meaning, and name this meaning change under ## Merge danger."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/burler/burler-cluster-fan.md:32
    summary: "The cluster and assist consolidation rules retag every kept finding origin lens, handler or assist and preserve only origin webster, so a hardener round run with hardener.reviewer_fan set loses origin focus, and the round check then refuses the undriven-focus finding outside the changed files."
    fix: "In burler-cluster-fan.md (consolidation line), burler-assist-cluster-fan.md and burler-assist-consolidation.md, extend the webster exception so a decision-class finding that records an undriven or WARN focus entry keeps origin: focus. Make the matching origin rule in burler-review-format.md name the same exception."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/roundcheck.go:325
    summary: "The origin focus exemption has no gate in the round context, unlike the webster one, which needs ctx.HaltReason; any incremental review of any segment, Code-Review included, may place any number of BLOCKING decision findings tagged origin focus outside the changed files, each of which now escalates to the parent."
    fix: "Add a RoundContext field (e.g. FocusFindings bool) that only a harden_round segment sets: thread it through shedadapters.RoundContextSources and BuildRoundContext from the harden-round Bouncer and BurlerReview wiring. Admit the origin: focus exemption in newFindings only when it is set. Add a TestCheckReviewRound row where a BLOCKING decision finding with origin: focus outside the changes is refused when the field is unset, and set the field on the existing passing row."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: assist:assist
    location: contracts/stencils/burler/burler-step-review.md:53
    summary: "The review step still says an incremental round raises a new finding only on a changed hunk, with no origin focus exception, and burler-review-format.md still opens its rule with sits on a changed hunk; a hardener reviewer whose undriven entry concerns unchanged source is told both to raise the finding there and not to."
    fix: "In burler-step-review.md's incremental-round bullet, add that a new BLOCKING decision-class finding with origin: focus may sit outside the changed hunks and files, beside the existing amending-card and webster exceptions. In burler-review-format.md, make the origin: focus exemption cover the changed-hunk rule as well as the file rule."
  - id: F6
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/shedadapters/bouncer_escalation_test.go:478
    summary: "The row BlockingEscalates below the budget leaves an open BLOCKING entry to the next round has a round-2 ledger that drops round 1's key alpha, so settleUnlessRefused rejudges it on the dropped-key check before settleUndecided runs; both paths return Stuck with no escalation, so the row passes without reaching the behaviour its name states."
    fix: "Give that row the ledger []evidenceEntry{{\"alpha\", \"resolved\", \"design\", \"MEDIUM\"}, {\"hard\", \"open\", \"scope\", \"BLOCKING\"}}, as commit f2cc07498 did for its two sibling rows, and assert that the round's judge outputs are still in place (not archived by a rejudge), so a rejudge can no longer satisfy the row."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/bouncer.go:1550
    summary: "The unreadable-ledger branch of blockingEscalationDetail, which escalates with cause budget and the could-not-be-read detail, has no test row."
    fix: "Add a TestBouncer_AutoAccept row with blockingEscalates: true, a spent budget and a round-2 ledger that does not parse, expecting Awaiting, cause EscalationBudget and a detail containing could not be read."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedrecipe/entries_bouncer_test.go:1431
    summary: "With blocking_escalates, a window opened by the hard cap alone now escalates with cause budget instead of hard-cap, and the only row that reaches it asserts Awaiting but not the cause."
    fix: "Assert the escalation cause (budget) on the row a hard cap of 1 offers only accept, or add a TestBouncer_AutoAccept row with the hard cap reached and the budget not spent that expects cause EscalationBudget."
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1515
    summary: "docs/overview.md and the CodeAutoAccept field comment in internal/loomengine/config.go still say code_auto_accept makes the perch accept a harvested round at a spent budget, with no carve-out for an open BLOCKING finding."
    fix: "In docs/overview.md:1515 and the field comment above DiscussionMaxBounces in internal/loomengine/config.go (line 520), add that on a perch whose Bouncer sets blocking_escalates a round with an open BLOCKING finding escalates to the run's parent instead of being accepted."
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/loomrecipe/decisionswitches_test.go#TestDecisionSwitches_SetOnTheRowsThatAskTheParent
    summary: "The redundancy report lists the test as contained by TestAmendmentTarget, which asserts only each recipe's amendment target row; the switch test pins the row sets of three recipe keys that coverage cannot see, so it is kept and needs the keep marker."
    fix: "Add //testtiming:keep pins which shipped recipe rows set escalate_decisions, decisions_escalated and blocking_escalates directly above func TestDecisionSwitches_SetOnTheRowsThatAskTheParent."
---

### [BLOCKING] A decision escalation's accept passes open BLOCKING findings at a spent budget without the parent

**Location:** internal/shedadapters/bouncerdecision.go#routeDecisions

**Issue:** `settleUnlessRefused` (`bouncer.go:1081`) runs `routeDecisions` before `settle`, so a Code-Review round whose review holds a decision finding never reaches the new `blockingEscalationDetail` check in `settleUndecided`.
It escalates with cause `decision`, which `arbiterstop.Eligibility` lets the arbiter settle.
The arbiter template lets it accept once the board entry and record answer the decision findings (`loom-template-arbiter.md:108-111`).
`settleDecisionAccept` then faces only `refusalLeavingOut`, which on a CONTINUE verdict checks dropped keys alone, and calls `acceptRound`.
`bouncer-fragment-decision.md` says so outright: "An `accept` passes the round over every BLOCKING finding listed".
The record's goal 2 decision is that a spent budget with a BLOCKING finding open "stops and asks its parent"; with both switches now on, this path skips the parent.
The scope and affected-set lenses found the same defect.

### [BLOCKING] `code_auto_accept` changes meaning with no reinterpretation

**Location:** internal/loomengine/template.yaml:53

**Issue:** The template comment changes from "accept … with its open findings, BLOCKING ones included" to "a round with an open BLOCKING finding is never accepted this way but escalates to the run's parent".
A hub that set `code_auto_accept: true` now gets a different effect.
`PATTERN-config-layers` says a meaning change ships a `configreg.Reinterpretation` that rewrites the value or flags it with a way forward, and none ships.
The change description argues that the recipe key carries the change and the setting "keeps its value and meaning", which the diff's own comment rewrite contradicts.
The rubric's merge-danger bullet also makes a `two-way` call a finding when the diff changes what a config setting means.
The operator's 2026-11-22 decision orders the behaviour; it does not waive the flag, and per the constraints the PATTERN entry wins.

### [MEDIUM] Consolidation drops `origin: focus`

**Location:** contracts/stencils/burler/burler-cluster-fan.md:32

**Issue:** The consolidation rule tags every kept finding `lens:<name>` or `handler`, and keeps only `origin: webster`.
The assist cluster and assist consolidation stencils do the same.
The hardener rows set `reviewer_fan_setting: hardener.reviewer_fan`, so a fanned hardener round rewrites the focus finding's origin.
`newFindings` then refuses it outside the changed files, and the reviewer can pass the round check only by dropping the undriven-focus finding, the gap goal 3 closes.
The assistant raised the same defect.

### [MEDIUM] The focus exemption is open to every segment

**Location:** internal/burlerengine/roundcheck.go:325

**Issue:** The webster exemption needs `ctx.HaltReason` and admits one finding.
The focus exemption needs only the finding's own `origin` and `class`, and `RoundContext` carries nothing that marks a hardener round.
A Code-Review reviewer may tag any number of out-of-change BLOCKING findings `origin: focus, class: decision`, and with `escalate_decisions` now on they each stop the run for the parent.
The focus directive asked whether the check admits a focus finding "without admitting unrelated ones"; it does not.

### [MEDIUM] The review step still requires every new incremental finding on a changed hunk

**Location:** contracts/stencils/burler/burler-step-review.md:53

**Issue:** The step's bullet "In an incremental round, raise a new finding only at `BLOCKING`, and only on a changed hunk" has exceptions for an amending card and `origin: webster`, but not for `origin: focus`.
`burler-review-format.md:76-78` exempts a focus finding from "the file rule" while keeping "sits on a changed hunk".
`harden-rubric-review.md` now tells the reviewer to locate the finding "at the module source the entry concerns", which in an incremental round is often unchanged.

### [MEDIUM] The below-budget BLOCKING row reaches the dropped-key rejudge

**Location:** internal/shedadapters/bouncer_escalation_test.go:478

**Issue:** The fixture writes round 1's ledger with key `alpha`, and this row's round-2 ledger holds only `hard`.
`droppedKeyRefusal` refuses the judgment and `rejudge` returns Stuck before `settleUndecided` runs, so the row's Stuck and no-escalation assertions pass on the wrong path.
Commit `f2cc07498` added `alpha` to the two sibling rows for this reason and missed this one.

### [LOW] The unreadable-ledger branch has no test

**Location:** internal/shedadapters/bouncer.go:1550

**Issue:** The commit message, the `BlockingEscalates` doc and the spec all state that an unreadable ledger escalates with cause `budget`.
No `TestBouncer_AutoAccept` row reaches that branch, and the mutation report lists two uncovered changed blocks.

### [LOW] The hard-cap-only window's new cause is not asserted

**Location:** internal/shedrecipe/entries_bouncer_test.go:1431

**Issue:** `windowOpen` includes `atHardCap`, so with `blocking_escalates` a hard-cap round with an open BLOCKING entry now escalates with cause `budget` (parent only) instead of `hard-cap`.
The one row that reaches it asserts only Awaiting, which the old `hard-cap` path also returned.

### [LOW] Two descriptions of `code_auto_accept` are stale

**Location:** docs/overview.md:1515

**Issue:** `docs/overview.md:1515` and the comment at `internal/loomengine/config.go:520` say auto-accept accepts a harvested round at a spent budget, with no BLOCKING carve-out.

### [NIT] The kept switch test needs its keep marker

**Location:** internal/loomrecipe/decisionswitches_test.go#TestDecisionSwitches_SetOnTheRowsThatAskTheParent

**Issue:** `TestAmendmentTarget` (`recipe_test.go:300-327`) asserts `AmendmentTarget`'s row name only.
The switch test asserts which rows carry each of three keys, which coverage blocks cannot see, so the rubric keeps it under `//testtiming:keep`.
The assistant raised the same point.

## Rejected

- Affected-set finding 1, "the arbiter cannot rule first on an `origin: focus` finding": `loom-template-arbiter.md:62-70` frees the arbiter from the parent's operator-decision-only rule, and its `decision` cause rule (lines 108-111) lets it record answers and accept or continue, so the arbiter can rule first.
- Assistant F1, "a held finding settles without its recorded ruling": the escalation is the gate, and the accept recorded by the parent (or by the arbiter, which records each answer first) is the parent's ruling; goal 1 asks that the run wait for it, which the Awaiting escalation does.
- No `arbiterstop` change is missing for goal 3: `Eligibility` already admits cause `decision` and refuses `budget` to the parent, as the scope lens and the change description state.
- The Loosened-Check trailer on `TestBouncer_SoftCapRule` holds: the dropped path no longer reaches the soft-cap rule, and `TestBouncer_AutoAccept`'s spent-budget BlockingEscalates rows assert the escalation that replaces it.
- The Root-Cause trailer names the three defects the diff removes, not their symptoms.
